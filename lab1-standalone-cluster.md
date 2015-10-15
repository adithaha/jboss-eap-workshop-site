# LAB1

Pada LAB ini kita akan membuat 2 server JBoss EAP yang akan diset sebagai standalone HA (high available), artinya masing-masing server akan berjalan selayaknya saat kita jalankan dengan menggunakan standalone.sh/.bat atau standalone-full.sh/.bat, masing-masing server akan memiliki admin console sendiri-sendiri artinya manajemen server, deployment aplikasi dilakukan secara independent, tetapi kedua server akan saling mereplikasi session data (HTTP session maupun EJB Session) dan kedua server akan dikenali oleh JBoss Web Server (dengan modul  mod_cluster) sebagai dua server yang identik dan JBoss Web Server akan berlaku sebagai load balancer yang membagi trafik ke kedua server tersebut.

Pada LAB ini kita akan men-setup hanya dua server di mesin yang sama, tapi pada dasarnya cara yang sama dapat kita terapkan untuk jumlah server lebih dari dua dan di mesin yang berbeda.

```
                                     ,------> JBoss EAP 
                                     |        (server1)
 Client  -----> JBoss Web Server ----+
(browser)        (loadbalancer)      |
                                     '------> JBoss EAP 
                                              (server2)

```


Langkah pertama untuk membuat 2 buah server yang terkonfigurasi HA adalah menginstall JBoss EAP.  Jika kita menggunakan physical machine yang berbeda maka kita perlu menginstal JBoss EAP di masing-masing mesin tersebut. 

Instalasi JBoss EAP
-------------------

Karena kita akan menggunakan mesin yang sama, maka instalasi hanya diperlukan sekali tapi nantinya kita akan jalankan 2 server dengan konfigurasi sendiri-sendiri. Masing-masing server yang jalan di mesin yang sama akan menggunakan port yang berbeda. Lain halnya jika kita menggunakan mesin yang berbeda, kita dapat menggunakan port yang sama (sebaiknya memang kita set menggunakan port yang sama untuk kemudahaan administrasi).

EAP sudah terinstall pada lab sebelumnya.


Mempersiapkan Dua Server EAP
----------------------------


1.  Buat dua folder konfigurasi untuk masing-masing server EAP.
   
    Jika anda menggunakan mesin yang sama, create dua direktori baru sebagai server1 dan server2, kemudian copy direktori `standalone/` ke kedua direktori tersebut.

	```sh
	d:
	cd /server/jboss/eap/
	mkdir server-standalone-ha
	cd server-standalone-ha
	mkdir server1
	mkdir server2
	cp -R jboss-eap-6.4/standalone server1
	cp -R jboss-eap-6.4/standalone server2
	```

2.  Edit file `standalone-ha.xml` pada masing-masing direktori server1 dan server2 yaitu di `serverX/configuration/` (ganti X dengan 1 atau 2)
    
    Cari konfigurasi __modcluster__ seperti dibawah ini. Lalu ganti IP address yang tertera pada attribute __`proxy-list`__. Karena kita akan menginstal JBoss Web Server (Load balancer) pada mesin yang sama (localhost) tadi kita isi kan IP 127.0.0.1 dengan port default yaitu 6666.
    

	```xml
	<subsystem xmlns="urn:jboss:domain:modcluster:1.1">
		<mod-cluster-config advertise-socket="modcluster" sticky-session="false" connector="ajp" proxy-list="127.0.0.1:6666">
		    <dynamic-load-provider>
		        <load-metric type="busyness"/>
		    </dynamic-load-provider>
		</mod-cluster-config>
	</subsystem>
	```

3.  Edit file `standalone-ha.xml` pada masing-masing direktori server1 dan server2 yaitu di `serverX/configuration/` (ganti X dengan 1 atau 2)
    
    Cari konfigurasi __jgroups__ seperti dibawah ini. Default adalah menggunakan UDP atau multicast, tidak perlu ada konfigurasi setiap host. Di sini kita akan mencoba menggunakan TCP atau unicast, dan konfigurasi tiap host diperlukan.

    Bisa kita lihat konfigurasi JGroups dibawah ini. Konfigurasi ini sudah sedikir dimodifikasi, versi aslinya anda tidak akan menemukan blok element `<stack name="tcp">`. Blok elemen ini digunakan jika komunikasi antar server dan dari server ke load balancer tidak bisa menggunakan multicast (UDP). Untuk menggunakan protokol TCP pada komunikasi cluster kita perlu mengubah nilai dari attribute `default-stack` pada element `subsystem` dibawah ini menjadi __`tcp`__

	```xml
        <subsystem xmlns="urn:jboss:domain:jgroups:1.1" default-stack="tcp">
            <stack name="udp">
                <transport type="UDP" socket-binding="jgroups-udp"/>
                <protocol type="PING"/>
                <protocol type="MERGE3"/>
                <protocol type="FD_SOCK" socket-binding="jgroups-udp-fd"/>
                <protocol type="FD"/>
                <protocol type="VERIFY_SUSPECT"/>
                <protocol type="pbcast.NAKACK"/>
                <protocol type="UNICAST2"/>
                <protocol type="pbcast.STABLE"/>
                <protocol type="pbcast.GMS"/>
                <protocol type="UFC"/>
                <protocol type="MFC"/>
                <protocol type="FRAG2"/>
                <protocol type="RSVP"/>
            </stack>
            <stack name="tcp">
                <transport type="TCP" socket-binding="jgroups-tcp"/>
                <protocol type="MPING" socket-binding="jgroups-mping"/>
                <protocol type="TCPPING">
                    <property name="initial_hosts">localhost[7600],localhost[7700]</property>
                    <property name="num_initial_members">2</property>
                    <property name="port_range"> 0</property>
                    <property name="timeout">
                        2000
                    </property>
                </protocol>
                <protocol type="MERGE2"/>
                <protocol type="FD_SOCK" socket-binding="jgroups-tcp-fd"/>
                <protocol type="FD"/>
                <protocol type="VERIFY_SUSPECT"/>
                <protocol type="pbcast.NAKACK"/>
                <protocol type="UNICAST2"/>
                <protocol type="pbcast.STABLE"/>
                <protocol type="pbcast.GMS"/>
                <protocol type="UFC"/>
                <protocol type="MFC"/>
                <protocol type="FRAG2"/>
                <protocol type="RSVP"/>
            </stack>
        </subsystem>
	```

    Jika anda menggunakan protocol TCP dan menggunakan mesin yang berbeda, ganti nilai element `property` dengan name `initial_host` sesuai IP address dari masing-masing mesin, misalnya `192.168.0.12[7600],192.168.0.13[7600]`.

    Nilai 7600 adalah port dari __jgroups-tcp__ yang dispesifikasikan pada element `socket-binding-group`

	
4.  create file untuk menjalankan server di direktori D:/server/jboss/eap/server-standalone-ha/serverX/run.bat
    
    ```sh
    ../../jboss-eap-6.4/bin/standalone.bat -b 0.0.0.0 -c standalone-ha.xml -Djboss.server.base.dir=standalone -Djboss.node.name=server1 -Djboss.socket.binding.port-offset=0
    ```
    
    ```sh
    ../../jboss-eap-6.4/bin/standalone.bat -b 0.0.0.0 -c standalone-ha.xml -Djboss.server.base.dir=standalone -Djboss.node.name=server2 -Djboss.socket.binding.port-offset=100
    ```
    
    Berikut penjelasan mengenai opsi yang digunakan pada perintah diatas
 

	* `-b` : Binding address. 0.0.0.0 artinya port akan di-binding kesemua network interface/IP address yang dimilimi mesin tersebut.
	* `-Djboss.node.name` : Nama node.
	* `-Djboss.socket.binding.port-offset` : Default port yang akan digunakan nilainya akan ditambahkan dengan nilai offset ini. Misalnya default port 8080 akan menjadi
	  8180 (8080+100) jika nilai offset adalah 100
	  
5.  Jalankan script run.bat pada kedua folder

### Melihat Konfigurasi HA dari server EAP

Sebelum kita lanjut dengan instalasi JBoss Web Server (Apache HTTP Server) dan module mod_clusternya. Saya akan bahas terlebih dahulu mengenai konfigurasi HA dari server EAP.

Kemudahan konfigurasi HA pada LAB ini karena sudah tersedianya file konfigurasi HA bawaan dari EAP yaitu file `standalone-ha.xml` atau `standalone-full-ha.xml`. Lalu apa bedanya file tersebut dengan file `standalone.xml` yang tanpa HA? Kita akan bandingkan sekarang. Silakan buka file `standalone-ha.xml` lagi

#### Extension Module

Untuk konfifurasi HA, dibutuhkan extensions tambahan berikut ini:

	```
    <extensions>
        <extension module="org.jboss.as.clustering.infinispan"/>
        <extension module="org.jboss.as.clustering.jgroups"/>
        ...
    </extensions>
    ```
    
[Infinispan]() adalah software untuk in-memory data grid yang digunakan JBoss EAP untuk menyimpan data HTTP/EJB session, cache

[JGroups](http://www.jgroups.org/) adalah software yang digunakan untuk komunikasi antar server/node dalam satu cluster

#### Replikasi Session

Replikasi session pada konfigurasi HA secara default sudah terkonfigurasi seperti dibawah ini. Replikasi session dilakukan dengan mode REPL artinya session akan direplikasi ke semua server dalam cluster. 

Keterangan mengenai detail konfigurasi bisa dilihat di [dokumentasi](https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/6.1/html/Development_Guide/chap-Clustering_in_Web_Applications.html#Enable_Session_Replication_in_Your_Application)

```xml
        <subsystem xmlns="urn:jboss:domain:infinispan:1.4">
             ...
            <cache-container name="web" aliases="standard-session-cache" default-cache="repl" module="org.jboss.as.clustering.web.infinispan">
                <transport lock-timeout="60000"/>
                <replicated-cache name="repl" mode="ASYNC" batching="true">
                    <file-store/>
                </replicated-cache>
                <replicated-cache name="sso" mode="SYNC" batching="true"/>
                <distributed-cache name="dist" l1-lifespan="0" mode="ASYNC" batching="true">
                    <file-store/>
                </distributed-cache>
            </cache-container>
            <cache-container name="ejb" aliases="sfsb sfsb-cache" default-cache="repl" module="org.jboss.as.clustering.ejb3.infinispan">
                <transport lock-timeout="60000"/>
                <replicated-cache name="repl" mode="ASYNC" batching="true">
                    <eviction strategy="LRU" max-entries="10000"/>
                    <file-store/>
                </replicated-cache>
                <replicated-cache name="remote-connector-client-mappings" mode="SYNC" batching="true"/>
                <distributed-cache name="dist" l1-lifespan="0" mode="ASYNC" batching="true">
                    <eviction strategy="LRU" max-entries="10000"/>
                    <file-store/>
                </distributed-cache>
            </cache-container>
            ...
        </subsystem>
```
> CATATAN: Agar web session direplikasi, aplikasi web yang dideploy perlu ditambahkan tag **`<distributable/>`** di file `web.xml` seperti ini
>
    
    <web-app  xmlns="http://java.sun.com/xml/ns/j2ee"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
          xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
                              http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd" 
          version="2.4">
        <distributable/>
    </web-app>
    

#### Socket Binding

Konfigurasi yang berbeda pada mode standalone HA (cluster) yang lain adalah pada blok socket binding. Pada konfigurasi ini terdapat konfigurasi untuk port yang digunakan untuk mekanisme clustering.

Coba buka kembali standalone-ha.xml, dan cari tag <socket-binding-group>. Element ini mendefinisikan semua port yang digunakan.


Instalasi dan Konfigurasi JBoss Web Server dan mod_cluster
----------------------------------------------------------

Untuk platform Windows lihat di sub-bab setelah ini.

>> PERHATIAN: Default port dari HTTP server mod_cluster yang didownload adalah 6666. Jika anda menggunakan browser Google Chrome, anda akan mendapatkan error __ERR_UNSAFE_PORT__ karena port tersebut dianggap tidak aman. Agar port tersebut tidak diblok maka anda dapat menambahkan opsi misalnya `–explicitly-allowed-ports=6666,6667` pada shortcut dari aplikasi Chrome.

Saat penulisan artikel ini, versi terakhir JBoss Web Server adalah versi 2.1.0 dan kita akan gunakan versi tersebut dalam LAB ini.

Download Apache HTTP Server dari JBoss Web Server versi Enterprise dari website [Red Hat](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?downloadType=distributions&product=webserver&productChanged=yes) sesuai dengan platform yang and a pakai. Untuk LAB ini silakan download file dari link __Red Hat JBoss Web Server 2.1.0 Apache HTTP Server for RHEL 7 x86_64__

Jika tidal memiliki Red Hat support subscription, anda bisa juga download versi community-nya di link berikut:

http://mod-cluster.jboss.org/downloads/1-2-6-Final-bin

Pilih platform yang sesuai

Mod_cluster di Windows
----------------------

Download mod_cluster untuk windows x86

[mod_cluster-1.2.6.Final-windows-x86.zip](http://downloads.jboss.org/mod_cluster//1.2.6.Final/windows/mod_cluster-1.2.6.Final-windows-x86.zip)

1. Ekstrak file `mod_cluster-1.2.6.Final-windows-x86-ssl.zip` di direktori --misalnya-- `D:/server/httpd-2.2`
2. Buka command prompt dengan user Administrator (run as Administrator)
3. Masuk ke direktori tersebut dan jalankan `installconf.bat`

	```
	D:\
	cd server\httpd-2.2\bin
    installconf.bat
	```
4. Jalankan Web Server dengan perintah `httpd.exe`
5. Modul mod_cluster secara default sudah terinstal dan terkonfigurasi sebagai modul Apache HTTP server. 
    Kita sekarang bisa cek JBoss EAP cluster yang sudah terdeteksi oleh  Apache HTTP server dengan mengakses 
    ke URL berikut:
    
    [http://localhost:6666/mod_cluster_manager](http://localhost:6666/mod_cluster_manager)

![Tampilan mod_cluster_manager](https://cloud.githubusercontent.com/assets/3068071/7254245/ff17c642-e86b-11e4-8206-2be8ddcdb2c8.png)

Test HA cluster
---------------

Kita akan test HA Cluster dengan beberapa scenario:

  - Load balancing HTTP request. Kita akan gunakan browser untuk mengakses suatu aplikasi ke IP & port dari HTTP server (load balancer). Kita harapkan load balancer akan mengarahkan request ke masing-masing server JBoss EAP secara seimbang (load balance)
  - Failed over HTTP request. Kita akan matikan salah satu server JBoss EAP lalu akses aplikasi IP & port dari HTTP server (load balancer). Kita harapkan load balancer dapat selalu mengarahkan request ke server yang hidup, sehingga kita tidak pernah mendapatkan error karena tidak dapat terkoneksi ke server yang dimatikan. 
  - Kita juga akan lihat bahwa session antar server JBoss EAP akan saling tereplikasi.


   
1. Deploy file `cluster-test.war` ke masing-masing server JBoss EAP dengan meng-copy file ke direktory `D:/server/jboss/eap/server-standalone-ha/serverX/standalone/deployment`

  ```
  D:/server/jboss/eap/server-standalone-ha/serverX/standalone/deployment
  ```

3. Test masing-masing aplikasi dengan cara mengakses langsung URL dari JBoss EAP (bukan URL load-balancer)
   Buka browser dan akses kedua URL berikut:
   
   - [http://localhost:8180/cluster-test](http://localhost:8180/cluster-test)
   - [http://localhost:8280/cluster-test](http://localhost:8280/cluster-test)
   
   Lihat tampilan yang muncul di browser, nama dari "nodeId" tiap server yang diakses berbeda yaitu "server1" dan "server2"
   Refresh masing-masing halaman tersebut, dan perhatikan jumlah session ("# of requests placed on session").
   
4. Test aplikasi dengan cara mengakses URL load-balancer yaitu 
   
   [http://localhost:6666/cluster-test](http://localhost:6666/cluster-test)
   
   Perhatikan lagi "nodeId" dan jumlah session.
   
5. Stop salah satu server, ctrl-c 
   
   Akses halaman [http://localhost:6666/cluster-test](http://localhost:6666/cluster-test) secara berkali-kali dan 
   perhatikan nama dari "nodeId" untuk melihat efek dari matinya server1.
   





