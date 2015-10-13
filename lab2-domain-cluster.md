# LAB: Cluster Domain

Pada LAB ini kita akan membuat 4 server JBoss EAP yang akan diset sebagai sebuah **server group** atau kita sebut **cluster**, dan ke-4 server tersebut dapat dikontrol & dimonitor oleh sebuah admin console terpusat yang berada di **Domain Controllerr**.

Ke-empat server akan saling mereplikasi session data (HTTP session maupun EJB Session), replikasi message pada messaging engine dan akan dikenali oleh JBoss Web Server (dengan modul  **mod_cluster**) sebagai server yang identik. JBoss Web Server akan berlaku sebagai load balancer yang membagi trafik dari pengguna aplikasi ke semua server tersebut.

Pada LAB ini kita akan mensimulasikan ke-4 server JBoss EAP tersebut dijalankan di dua mesin terpisah. Jadi masing-masing mesin akan menjalankan 2 server instance JBoss EAP. Sedangkan JBoss Web Server dan JBoss EAP yang berfungsi sebagai Domain Controller disimulasikan berjalan di mesin lain. 
Total mesin fisik yang kita simulasikan ada 4, tetapi dalam LAB ini kita hanya akan menggunakan satu mesin PC atau laptop saja.

Setelah menyelesaikan LAB ini diharapkan anda dapat menangkap konsep kerja dan cara konfigurasi JBoss EAP sehingga dapat melakukan setup untuk jumlah server yang lebih sedikit atau lebih banyak sesuai kebutuhan dan juga bisa membuat server-group lain. Sebuah Domain Controller dapat mengelola beberapa server-group atau cluster.

Berikut gambar arsitektur dari apa yang akan kita setup:


```
                                           +-----------+
                                  ,------->| JBoss EAP | 
                                  |        | (server1) |
                                  |        |           |.......
                                  |        |           |      .
                                  +------->| JBoss EAP |      .
                                  |        | (server2) |      .    +---------------------+
                                  |        +-----------+      .    |      JBoss EAP      |
             +-----------------+  |          machine-2        .....| (Domain Controller) |
 Client  --->| JBoss Web Server|--+                           .    +---------------------+
(browser)    | (loadbalancer)  |  |                           .          machine-1
             +-----------------+  |        +-----------+      .
                  Mesin-Z         +------->| JBoss EAP |      .
                                  |        | (server3) |      .
                                  |        |           |.......
                                  |        |           |
                                  '------->| JBoss EAP |
                                           | (server4) |
                                           +-----------+
                                             machine-3


```
Pada Mesin-A dan Mesin-B akan dijalankan masing-masing 2 server instance JBoss EAP untuk memperlihatkan kemampuan vertical scalability. Dan semua server JBoss EAP tersebut merupakan satu kesatuan (group atau cluster) sehingga memperlihatkan kemampuan horizontal scalability.

Langkah-langkah besar untuk mensetup lingkungan (environment) seperti tergambar diatas yang akan kita jalankan sebagai berikut:

  1. Setup Domain Controller di Mesin-X
  2. Setup JBoss EAP di Mesin-A
  3. Setup JBoss EAP di Mesin-B
  4. Deploy aplikasi ke semua server JBoss EAP di Mesin-A dan Mesin-B lewat Domain Controller
  5. Test akses aplikasi 
  6. Setup Load Balancer/Web Server di Mesin-Z
  7. Test akses applikasi


LAB: Menjalankan Domain dengan topology default
================================================

File Konfigurasi Domain
-----------------------

JBoss EAP sudah memberikan contoh file-file konfigurasi yang bisa langsung dijalankan sehingga memudahkan kita untuk melakukan setup domain. Tentu saja untuk kebutuhan nyata di lingkungan production, file konfigurasi yang ada perlu dimodifikasi sesuai kebutuhan.

File-file konfigurasi untuk setup domain ada di direktori `domain` didalam direktori dimana JBoss EAP diinstal. Isi dari direktori tersebut adalah sebagai berikut:

	```
	application-roles.properties           
	application-users.properties
	mgmt-groups.properties
	mgmt-users.properties	
	default-server-logging.properties
	domain.xml
	host-master.xml
	host-slave.xml
	host.xml
	logging.properties

	```

 - File-file dengan extension `.properties` adalah file yang menyimpan informasi credential (username, group, dan password). 
 - File `domain.xml` adalah file konfigurasi domain yang mendefinisikan module-module yang digunakan pada domain dan **mendefinisikan server-group atau cluster**
 - File `host-master.xml` adalah contoh file konfigurasi untuk sebuah mesin yang kita set sebagai Domain Controller, yaitu node yang menjadi sentral pengaturan dan tidak menjalankan server yang bekerja menerima request dari pengguna aplikasi 
 - File `host-slave.xml` adalah contoh file konfigurasi untuk sebuah mesin yang kita set sebagai application server yang akan kita deploy aplikasi dan bekerja menerima request dari pengguna aplikasi.
 - File `host.xml` adalah contoh file konfigurasi untuk sebuah mesin yang akan kita set sebagai Domain Controller tapi juga akan menjalankan application server yang akan menjalankan aplikasi.

Sebelum kita mencoba menbuat environment seperti gambar diatas, kita akan coba dulu eksplor contoh konfigurasi tersebut dengan menjalankannya __tanpa modifikasi sedikitpun__.

Kita dapat menjalankan sebuah lingkungan cluster dengan arsitektur seperti ini dengan hanya satu perintah `domain.sh` dari `<direktori_installasi_jboss_eap>/bin/`. Semua komponen yang tergambar di arsitektur dibawah ini jalan di satu mesin.


```     
                                    ,----------------------.
                                    |                      |
                                    |  +----------------+  |
                                +----->|   JBoss EAP    |  |
                                |   |  | (server-one)   |  |
+----------------------+        |   |  +----------------+  |
|      JBoss EAP       |--------+   |                      |
| (Domain Controller)  |---+    |   |  +----------------+  |
+----------------------+   |    |   |  |  JBoss EAP     |  |
                           |    +----->|  (server-two)  |  |
                           |        |  +----------------+  |
                           |        |                      |
                           |        '----------------------'
                           |            server-group:
                           |         (main-server-group)
                           |
                           |        ,----------------------.
                           |        |  +----------------+  |
                           +---------->|   JBoss EAP    |  |
                                    |  | (server-three) |  |
                                    |  +----------------+  |
                                    '----------------------'
                                           server-group:
                                       (other-server-group)
                                       
                                       
    Semua node tersebut jalan di satu host (mesin)
```

Sekarang ikuti langkah-langkah berikut untuk menjalankan JBoss EAP dengan arsitektur tersebut dan mengeksplor konfigurasi serta prosesnya.

1. Jalankan perintah `domain.sh`  tunggu sampai selesai proses booting dengan output terakhir seperti ini: 

    ```
    [Server:server-two] 11:55:39,476 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015874: JBoss EAP 6.3.0.GA (AS 7.4.0.Final-redhat-19) started in 7883ms - Started 184 of 221 services (64 services are lazy, passive or on-demand)
    ```
    
   Perintah tersebut akan membaca file konfigurasi `domain.xml` dan `host.xml` di folder `domain`
   
2. Akses admin cosnole: [http://127.0.0.1:10190/](http://127.0.0.1:10190/)

3.  Lalu klik menu "Domain" > "Overview" atau akses langsung ke URL: 
    [http://127.0.0.1:10190/console/App.html#server-groups](http://127.0.0.1:10190/console/App.html#topology)

	Kita bisa lihat pada halaman tersebut beberapa server-group dan masing-masing server yang masuk dalam server-group tersebut dan juga kita bisa lihat status dari masing-masing server apakah jalan atau mati.

	> Pada EAP versi 6.3 tabel tolopology tersebut dapat diakses di menu "Runtime" lalu pilih tab TOPOLOGY

	![default server group](https://cloud.githubusercontent.com/assets/3068071/7273979/14bac12c-e923-11e4-8ee0-e5389cd0f145.png)
   
    Lihat di halaman tersebut ada 2 server-group, dengan 3 EAP servers seperti yang terlihat gambar arsitektur diatas.
   
    Kedua server di group `main-server-group` yaitu `server-one` dan `server-two` dalam keadaan hidup (jalan), karena diset otomatis jalan ketika domain dijalankan. Sedangkan `server-three` dalam keadaan mati karena memang diset untuk tidak otomatis jalan.
   
4.  Arahkan mouse pointer ke masing-masing server dalam tabel tersebut. Kita akan lihat link "Stop server" dan "Force  shutdown" untuk server yang jalan dan kita lihat link "Start server" untuk server yang dalam keadaan mati.

	Coba nyalakan `server-three` dengan mengklik link "Start server", lalu klik tombol "Confirm" pada dialog window.
   
5. Lihat file konfigurasi `domain.xml` yang ada di `<direktori_installasi_jboss_eap>/domain`, dua server-group dengan nama `main-server-group` dan `other-server-group` dapat dilihat di file tersebut dibagian paling bawah:

	```xml
	<server-groups>
	    <server-group name="main-server-group" profile="full">
	        <jvm name="default">
	            <heap size="1000m" max-size="1000m"/>
	            <permgen max-size="256m"/>
	        </jvm>
	        <socket-binding-group ref="full-sockets"/>
	    </server-group>
	    <server-group name="other-server-group" profile="full-ha">
	        <jvm name="default">
	            <heap size="1000m" max-size="1000m"/>
	            <permgen max-size="256m"/>
	        </jvm>
	        <socket-binding-group ref="full-ha-sockets"/>
	    </server-group>
	</server-groups>   
	```
	
6. Pada web admin console, klik menu "Domain" > "Server Configurations". Kita bisa lihat ada 3 server disitu yaitu `server-one`, `server-two`, dan `server-three`. 

7. Konfigurasi ke-3 server tersebut dapat dilihat di file `host.xml` yang ada di direktori `<direktori_installasi_jboss_eap>/domain`

	```
	    <servers>
	        <server name="server-one" group="main-server-group">
	            <socket-bindings port-offset="200"/>
	        </server>
	        <server name="server-two" group="main-server-group" auto-start="true">
	            <socket-bindings port-offset="350"/>
	        </server>
	        <server name="server-three" group="other-server-group" auto-start="false">
	            <socket-bindings port-offset="450"/>
	        </server>
	    </servers>
	```
	masing-masing server harus diset `port-offset`-nya karena ketiga server tersebut akan jalan di host yang sama sehingga port yang digunakan tidak boleh sama.

8. Eksplor proses Java yang dijalankan oleh JBoss EAP dengan perintah `ps ax| grep jboss`. Hasilnya anda akan lihat ada 4 proses Java seperti dibawah ini:

	```
	5770 s000  S+     0:01.31 /path/to/1.6.0.jdk/bin/java -D[Process Controller] ...
	5771 s000  S+     0:06.59 /path/to/1.6.0.jdk/bin/java -D[Host Controller] ...
	5773 s000  S+     0:13.97 /path/to/1.6.0.jdk/bin/java -D[Server:server-one] ...
	5774 s000  S+     0:13.93 /path/to/1.6.0.jdk/bin/java -D[Server:server-two] ...
	5775 s000  S+     0:13.93 /path/to/1.6.0.jdk/bin/java -D[Server:server-three] ...
    ```

    Dari list proses tersebut kita bisa melihat terdapat proses-proses Java berikut:
    
	- Process Controller: Proses ini akan dijalan pertama kali, kemudian proses ini akan menjalankan dan memastikan proses 'Host Controller' terus jalan. Jika proses Host Controller dimatikan, Proses Controller akan otomatis menghidupkannya lagi.
	- Host Controller: Proses ini adalah proses yang bertanggung jawab untuk komunikasi antara Domain Controller dengan JBoss EAP server instance yang jalan di mesin yang sama. Biasanya di setiap mesin yang akan kita setup sebagai bagian dari cluster akan memiliki satu Host Controler.
	- Server: Proses ini adalah JBoss EAP server yang akan menjalan aplikasi. 
	
9. SELESAI. Sekarang stop semua proses dengan menekan tombol CTRL+C pada terminal yang menjalankan `domain.sh`

LAB: Memulai membuat konfigurasi domain
=======================================

Sekarang mari kita mulai untuk membuat konfigurasi sesuai dengan contoh kasus yang digambarkan diawal artikel ini.

## Menyiapkan Domain Controller (MASTER) 

Kita akan mensimulasikan sebuah Domain Controller yang dijalankan di Mesin-X. Pada mesin ini tidak ada server EAP yang akan menjalankan aplikasi, hanya sebuah proses Domain Controller yang akan menjadi node untuk management dimana Management Console dijalankan. 

Langkah berikut menggunakan asumsi JBoss EAP anda diinstal di direktori `D:/server/jboss/eap/jboss-eap-6.4`

1.  Buat direktori `server-domain` di `D:/server/jboss/eap/`
2.  Buat direktori `machine-1` di `D:/server/jboss/eap/server-domain`, machine-1 akan digunakan untuk domain controller
3.  Buat direktori `machine-2` di `D:/server/jboss/eap/server-domain`, machine-2 akan digunakan untuk node server
4.  Buat direktori `machine-3` di `D:/server/jboss/eap/server-domain`, machine-3 akan digunakan untuk node server
5.  Copy direktori `D:/server/jboss/eap/jboss-eap-6.4/domain` ke `machine-1`, `machine-2` dan `machine-3`
6.  Buka file `domain.xml` di direktori `machine-1/domain/configuration/` dengan file editor. Di file ini kita akan melihat ada beberapa konfigurasi subsystem untuk masing-masing **profile** 

	```
	<profiles>
        <profile name="default">
        ...
        </profile>
        <profile name="ha">
        ...
        </profile>
        <profile name="full">
        ...
        </profile>
        <profile name="full-ha">
        ...
        </profile>
	</profiles>
	```

	Kemudian cari element `hornetq-server` di profile `ha` dan `full-ha`. Kita perlu menambahkan konfigurasi untuk hornetq engine pada konfigurasi cluster seperti ini:
	
	```
	<hornetq-server>
		<clustered>true</clustered>
	        <cluster-user>jms-user</cluster-user>
	        <cluster-password>simple-pass</cluster-password>
		...
	</hornetq-server>
	```
	
7.  create file untuk menjalankan server di direktori D:/server/jboss/eap/server-domain/server1/run.bat 
    ```
    ../../jboss-eap-6.4/bin/domain.sh -c domain.xml --host-config=host-master.xml -Djboss.domain.base.dir=domain -Djboss.node.name=server1 -Djboss.socket.binding.port-offset=0
    ```

8.  Jalankan Domain Controller atau master host dengan perintah berikut:

    ```
    run.bat
    ```
    
    Perintah tersebut akan menjalankan Process Controller dan Host Controller tanpa menjalankan satupun Server karena berbeda dengan file konfigurasi host default yaitu `host.xml`, file host yang digunakan pada perintah tersebut yaitu `host-master.xml` tidak memiliki element `<servers>...</servers>`

9.  Buka file `host-master.xml` dengan editor atau file viewer, perhatikan disitu tidak ada elemen `<servers>...</servers>`
    Lihat juga bahwa file ini memiliki element

	```
	<domain-controller>
       <local/>
    </domain-controller>
	```
	yang artinya, ketika kita jalankan perintah diatas maka host ini akan menjadi Domain Controller.
	
	Perhatikan juga di file tersebut didefinisikan 2 server group yaitu `main-server-group` dan `other-server-group` dengan profile yang berbeda.

10.  Eksplor proses Java yang ada pada sistem dengan perintah `ps ax |grep jboss`
	Anda akan mendapatkan 2 proses berikut
	```
	...java -D[Host Controller]...
	...java -D[Process Controller]...
	```

11.  Eksplor web management console: [http://127.0.0.1:9990/](http://127.0.0.1:9990/)
    [EAP v6.4] Klik menu Domain > Overview dan lihat pada tab TOPOLOGY, perhatikan tidak ada server atau server group yang terlihat di halaman tersebut.

    Lihat juga pada Domain > Server Groups, ada dua server groups di halaman tersebut seperti yang terlihat di `domain.xml`
   
    >> [EAP v6.3] dapat dilihat dimenu Domain dan Runtime > tab TOPOLOGY. 

	Port web management console tersebut didefinisikan di `host-master.xml` pada baris berikut:
	
	```
	 <http-interface security-realm="ManagementRealm">
        <socket interface="management" port="${jboss.management.http.port:9990}"/>
    </http-interface>
	```
    
    Nilai `${jboss.management.native.port:9999}` artinya port yang digunakan (default) adalah 9999 jika tidak dioveride dengan cara dispesifikasikan pada command line dengan opsi `-Djboss.management.native.port=XXXX` dimana XXXX adalah nilai yang akan menggantikan nilai 9999

12.  SELESAI. Kita sudah menyiapkan sebuah Domain Controller pada server1.


*  Nantinya semua EAP server yang ada di machine-2 dan machine-3 akan mengakses (tergabung ke) Domain Controller di machine-1 ini melalui port 9990 yaitu port yang didefinisikan sebagai __native management port__

	```
	<native-interface security-realm="ManagementRealm">
        	<socket interface="management" port="${jboss.management.native.port:9999}"/>
    	</native-interface>
	 ```


## Meyiapkan JBoss EAP Server (SLAVE)

Kita akan mensimulasikan penyiapan JBoss EAP Server di 2 mesin yaitu machine-2 dan machine-3 yang berbeda dengan mesin dimana dijalankan Domain Controller.


1.  Sebelumnya direktori machine-2 dan machine-3 sudah kita buat.
2.  Edit file `host-slave.xml` yang ada di direktori `machine-2`
    
    ```
      <host name="machine-2" xmlns="urn:jboss:domain:1.6">
    ``` 

    Pada file ini didefinisikan alamat IP atau hostname dan port dari host-master atau Domain Controller, anda bisa lihat seperti ini. 
   
    ```
    <management-interfaces>
            <native-interface security-realm="ManagementRealm">
                <socket interface="management" port="${jboss.management.native.port:19999}"/>
            </native-interface>
        </management-interfaces>
    ```
    
    Ubah port management dari machine-2 menjadi 19999.
    
3.  Ulangi langkah #2 untuk direktori `machine-3`, gunakan port management 29999.
4.  create file untuk menjalankan server di direktori D:/server/jboss/eap/server-domain/machine-2/run.bat

    ```
    ../../jboss-eap-6.4/bin/domain.sh --host-config=host-slave.xml -Djboss.domain.base.dir=domain -Djboss.domain.master.address=localhost
    ```
    
    create file untuk menjalankan server di direktori D:/server/jboss/eap/server-domain/machine-3/run.bat
    
     ```
    ../../jboss-eap-6.4/bin/domain.sh --host-config=host-slave.xml -Djboss.domain.base.dir=domain -Djboss.domain.master.address=localhost
    ```
    
    >> Pada kondisi nyata dimana Domain Controler atau master-host berbeda mesin dengan mesin anggota cluster maka nilai 127.0.0.1 harus diubah dengan IP address dari master-host.


   
7.  Tes dengan men-deploy aplikasi `cluster-test.war`. Buka management console, klik menu "Deployment", klik tombol "Add" dan klik "Browse" untuk memilih file `cluster-test.war`. Klik Next kemudian Save. 

8.  Pilih `cluster-test.war` di tabel content repository, lalu klik tombol "Assign". Pada window "Select server group" pilih `other-server-group` yaitu group yang profile-nya adalah `full-ha`. Klik Save.

9.  Pilih menu Deployment jika belum berada di halaman Deployment. Klik tab "SERVER GROUPS", kemudian pada tabel daftar server groups klik "View >" pada baris `other-server-group`. Anda bisa lihat daftar aplikasi yang sudah ter-deploy di `other-server-group`.

10. Test aplikasi dengan mengaksed ke URL berikut
    - server-two pada port (8080+150) -> [http://localhost:8230/cluster-test](http://localhost:8230/cluster-test)
    - server-four pada port (8080+300) -> [http://localhost:8380/cluster-test](http://localhost:8380/cluster-test)
   
## Menyiapkan Load Balancer (Apache Web Server dengan mod_cluster add-on)

Silakan dicoba sendiri, caranya hampir sama dengan pada saat anda mensetup untuk standalone-ha di LAB sebelumnya :-)

![image](https://cloud.githubusercontent.com/assets/3068071/7278786/cec78d8e-e941-11e4-80e0-5c2a49941e72.png)


