## LAB 01 - Installasi JBoss EAP 6

Pastikan anda sudah memiliki (terinstal JDK atau JRE) Java 1.6 atau yang lebih tinggi
Buka Terminal (Linux) atau Command Line (Windows) kemudian jalankan perintah
   
	```
   	java -version
   	```
   
Kemudian juga cek apakah variabel `JAVA_HOME` sudah diset di environment, jika belum set JAVA_HOME
   
  	```
   	export JAVA_HOME=/path/to/jdk
   	```
   	

Installasi Menggunakan GUI
==========================

1.  Jalankan perintah berikut untuk memulai instalasi menggunakan mode GUI

	```
	java -jar jboss-eap-6.4.0-installer.jar
	```

2.  Pilih bahasa yang digunakan selama instalasi, klik OK
3.  Setujui EULA, klik Next
4.  Saat diminta memasukan "installation path" ketik `/home/jboss/EAP-6.4`, klik Yes.
	Jika direktori belum ada, anda akan ditanya apakah akan dibuat direktori tersebut, klik OK.	
5.  Langkah selanjutnya akan ditampilkan paket-paket yang akan diinstal. Buka (expand) semua komponen yang ada di daftar sehingga anda bisa melihat semua komponen yang akan di instal.
	Klik Next.
6.  Saat diminta untuk membuat user, masukan `admin` sebagai username dan `password1!` sebagai password. Atau gunakan password yang lain sesuai keinginan anda
7.  Pilih Yes saat ditanya instalasi QuickStarts untuk instalasi contoh-contoh aplikasi yang bisa dideploy di JBoss EAP. Untuk environment production, anda tidak perlu instalasi QuickStarts. Klik Next 
8.  Langkah selanjutnya adalah mengatur konfigurasi Maven Repository.
    Maven diperlukan sehingga anda dapat melakukan build project yang ada di QuickStarts
9.  Langkah selanjutnya adalah menentukan port-port yang digunakan. Ada dapat memilih
	- Menggunakan default ports
	- Menentukan nilai port offset untuk semua default ports. Artinya nilai default port akan ditambahkan nilai offset.
	- Menuntukan sendiri nilai port yang akan digunakan. (Custom)
	
	Pilih opsi yang ke-3 dan pilih (checked) mengkonfigurasi port standalone dan domain. 
	
	Pada daftar port anda akan melihat pengaturan "System Property". Beberapa port yang umumnya sering kita ganti memiliki System Property yaitu nama variabel yang digunakan saat kita ingin mengganti nilai port tersebut pada saat menjalankan Jboss EAP dengan menggunakan command argument. 
	
	Misalnya kita ingin menggunakan port management-http yang berbeda, katakanlah port 9991, saat menjalankan EAP sehingga akan meng-overide setting yang ada di file konfigurasi, maka kita bisa menjalankan dengan perintah `standalone.sh -Djboss.management.http.port=9991`
	
	Klik Next beberapa kali sampai wizard port setting selesai.
		
10. Saat sudah sampai pada langkah "Server Launch", pilih  "Don't start the Server". Klik Next
11. Pada langkah "Logging Options", pilih No, untuk membiarkan pengaturan logging menggunakan konfigurasi default. Klik Next
12. Pada langkah "Configure Runtime Environment", pilih No. Klik Next dua kali sampai proses instalasi dimulai

Uninstall JBoss EAP
===================

Pada dasarnya anda dapat melakukan uninstall JBoss EAP hanya dengan menghapus direktori.

1.  Masuk ke direktori `/home/jboss/EAP-6.4/Uninstaller` dan jalankan program uninstaller

	```
	cd /home/jboss/EAP-6.4/Uninstaller
	java -jar uninstaller.jar
	```
2.  Pilih force the deletion..., klik Uninstall
3.  Klik Quit
4.  Lihat apakah direktori tempat instalasi EAP masih ada?

Installasi Menggunakan Mode Console
===================================

1.  Jalankan perintah berikut untuk memulai instalasi menggunakan mode console:

	```
	java -jar jboss-eap-6.4.0-installer.jar -console
	```
	
2. Ikuti perintah yang ada diconsole sampai beberapa step supaya anda tau saja cara instalasi ini. 

> Anda tidak perlu menyelesaikan cara instalasi ini, kapanpun sebelum proses instalasi selesai, cancel dengan menekan tombol CTRL+C, karena kita akan menginstall dengen menggunakan installer dalam bentuk file ZIP.
   


Eksplorasi JBoss EAP
====================
	
1. Lihat file log dengan menggunakan editor atau perintah `tail`. 

	```
    tail -f /home/jboss/EAP-6.4/standalone/log/server.log
	```
	
	Keluar dari perintah `tail` dengan CTRL+C
	

2. Shutdown dengan perintah `kill` atau jika menggunakan Windows anda bisa stop proses (end task) dari Window Task Manager/

	```
	kill -9 <PROCESS_ID>
	```
	
	Dimana PROCESS_ID adalah nomor proses yang ada di kolom ke-dua pada output perintah `ps`,
	pada contoh diatas PROSESS_ID adalah `1237`
	
	Jika hanya ada satu proses java, anda bisa menstop proses dengan perintah `killall java`

3. Masuk ke direktori `/home/jboss/EAP-6.4/bin` yang berisi beberapa script untuk menjalankan JBoss EAP, mengakses ke command line interface (CLI), dan juga direktori `init.d` yang menyimpan script untuk menjalankan JBoss EAP sebagai service di Linux.

	Buka dan lihat file `standalone.conf` atau di Windows file tersebut bernama `standalone.conf.bat`, lihat baris yang menunjukan setting minimum dan maximum heap size: 
	
	`JAVA_OPTS="-Xms1303m -Xmx1303m -XX:MaxPermSize=256m -Djava.net.preferIPv4Stack=true"`
	
	Kita dapat mengubah nilai Xms, Xmx dan XX:MaxPermSize sesuai kebutuhan.
	
4.  Masuk ke direktori `/home/jboss/EAP-6.4/modules`, anda akan mendapatkan banyak sekali direktori yang berisi module.
	
	Sebuah modul biasanya disimpan dalam organisasi direktori yang strukturnya mengikuti Java package, misalnya module untuk dom4j disimpan di direktori `<EAP_INSTALL_DIR>/modules/system/layers/base/org/dom4j/`. Di dalam direktori tersebut terdapat folder `main` dengan isi library (file JAR) dan metadata dari module tersebut yaitu file `module.xml`

	Berikut contoh isi file `module.xml`
	
	```
      <?xml version="1.0" encoding="UTF-8"?>
      <module xmlns="urn:jboss:module:1.1" name="org.dom4j">
        <properties>
          <property name="jboss.api" value="unsupported"/>
        </properties>
        <resources>
          <resource-root path="dom4j-1.6.1.redhat-6.jar"/>
              <!-- Insert resources here -->
        </resources>
        <dependencies>
            <module name="javax.api"/>
            <module name="com.sun.xml.bind"/>
            <module name="javax.xml.bind.api"/>
            <module name="org.jaxen"/>
        </dependencies>
      </module>	
	
	```
    
5.  Masuk ke direktori `welcome-content`, di direktori ini terdapat file-file default yang akan ditampilkan jika kita mengakses halaman web (default halaman web adalah http://localhost:8080)



   

   
