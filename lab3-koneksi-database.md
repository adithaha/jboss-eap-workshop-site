# Mengkonfigurasi Koneksi JBoss EAP ke ProsgreSQL Database

>> Tutorial ini dibuat dengan menggunakan JBoss EAP versi 6.4.0, cara yang sama untuk koneksi ke database lainnya misalnya
MySQL, Oracle Database, MariaDB akan sama saja. Yang membedakan adalah library JDBC, connection URL, driver class, dan 
datasource class.

Sebelum melakukan konfigurasi JBoss EAP agar terkoneksi ke database, pertama tentu perlu dipastikan bahwa database sudah
terinstall dan dapat diakses dari server EAP. 

Untuk dapat terkoneksi ke database, biasanya aplikasi menggunakan __datasource__ yang diakses menggunakan __JNDI__.
Datasource tersebut memberikan koneksi database kepada aplikasi. Datasource menggunakan __connection-pool__ yang mengelola
koneksi ke database agar lebih efisien. Tiap objek koneksi yang ada di connection-pool menggunakan __JDBC driver__ untuk 
membuatkoneksi ke database. Setiap database memiliki JDBC driver-nya masing-masing dan datasource perlu diset agar objek 
koneksi bisa dibuat saat dibutuhkan.

Konfigurasi datasource bisa dilihat dan dibuat lewat Web Management Console di menu Configuration > Connector - Datasources

<img src="https://cloud.githubusercontent.com/assets/3068071/7330740/a4b9ab42-eb1e-11e4-9b93-91aadcef55f8.png" height="500" width="500" >


Datasource membutuhkan JDBC driver, suatu JAR file yang bisa dikenali oleh EAP dengan dua cara yaitu:

 1. Men-deploy JDBC JAR file sebagai aplikasi. Cara ini lebih direkomendasikan karena lebih mudah dan dapat dengan mudah dilakukan pada arsitektur domain dimana memiliki banyak node di banyak host.
    
    Untuk dapat mendeploy JDBC JAR file, maka didalam JAR file diperlukan sebuah file bernama `java.sql.Driver` didalam direktori `META-INF/`. Isi file tersebut adalah fully qualified class name dari JDBC driver tersebut, misalnya untuk PostgreSQL adalah `org.postgresql.Driver` sedangkan untuk Oracle DB adalah `oracle.jdbc.OracleDriver`

    >> Biasanya sebuah JDBC driver type-4, yang biasa digunakan sudah memiliki file tersebut, sehingga sudah bisa langsung kita deploy ke JBoss EAP tanpa perlu perubahan.

 2. Membuat JDBC sebagai module di direktori `module/`. Contoh struktur direktori sebuah module JDBC driver adalah seperti berikut:

	```
	modules
	│
	├── postgresql
       └── main
           ├── module.xml
           └── postgresql-9.3-1102.jdbc4.jar
    ```

Ada beberapa cara untuk membuat atau mengkonfigurasi datasource di JBoss EAP, yaitu

1. Cara manual dengan membuat module JDBC driver dan mengubah file konfigurasi XML.
2. Dengan JBoss Command Line Interface (CLI). 
3. Menggunakan web management console

## Buat Module JDBC Driver di JBoss EAP

Download PostgreSQL JDBC driver dari https://jdbc.postgresql.org/download.html
Jika kita menggunakan Java 1.7 atau maka download file driver JDBC41, sedangkan jika JRE yang digunakan adalah versi 1.6
download file driver JDBC4.

1. Buat direktori `<EAP_INSTALL_DIR>/modules/postgresql/main` untuk menyimpan file JDBC library yang akan di-load oleh EAP sebagai module. 

	```
	cd <EAP_INSTALL_DIR>/modules
	mkdir postgresql/main
	```

2. Simpan file JDBC driver, misalnya `postgresql-9.3-1102.jdbc4.jar` di folder tersebut.
   Buat file `module.xml` di folder tersebut yang isinya seperti ini:

	```xml
	<?xml version="1.0" encoding="UTF-8"?>  
	<module xmlns="urn:jboss:module:1.0" name="org.postgresql">  
	 <resources>  
	 <resource-root path="postgresql-9.3-1102.jdbc4.jar"/>  
	 </resources>  
	 <dependencies>  
	 <module name="javax.api"/>  
	 <module name="javax.transaction.api"/>  
	 </dependencies>  
	</module>
	```
	
   Karakter pertama pada file tersebut tidak boleh spasi atau karakter lainnya tapi harus dimulai dengan `<?xml`

   Selai kita membuat module untuk JDBC driver.

## Buat Konfigurasi Datasource

3. Sekarang kita buat konfigurasi datasource dengan cara manual. Asumsi anda akan menjalankan Jboss EAP dalam mode standalone, maka ubah file konfigurasi EAP yang digunakan (`standalone.xml`). 
   
    Pada elemen `datasources` ubah konfigurasi datasource dengan `pool-name` ExampleDS menjadi seperti berikut:

	```xml
	<datasource jndi-name="java:jboss/datasources/ExampleDS" pool-name="ExampleDS" enabled="true" 
	            use-java-context="true" spy="true">
	    <connection-url>jdbc:postgresql://localhost:5432/bpms</connection-url>
	    <driver-class>org.postgresql.Driver</driver-class>
	    <driver>postgresql</driver>
	    <security>
	        <user-name>postgres</user-name>
	        <password>password</password>
	    </security>
	</datasource> 
	```

    Tambahkan elemen dibawahnya

	```xml
	<drivers>
	    <driver name="postgresql" module="org.postgresql">
	        <xa-datasource-class>org.postgresql.xa.PGXADataSource</xa-datasource-class>
	    </driver>
	    ...
	</drivers>
	```

    Selain cara manual diatas, kita juga bisa membuat datasource dari Web Management Console, dari menu 
    Configuration > Connection > Datasource.

4. Start atau restart JBoss EAP 
   
    Jika JBoss EAP belum dijalankan, maka anda bisa menjalankannya dengan perintah `standalone.sh` (Linux)
    dari direktori `<EAP_INSTALL_DIR>/bin/`
    
5.  buka web console di http://localhost:9990/

6.  Eksplorasi fields yang bisa diatur untuk sebuah datasource.
   
    Klik tab Attributes, Connection, Pool, Validation dan Timeouts.
    
    Pada masing-masing tab tersebut, klik link "Need Help?" untuk membaca pejelasan dari masing-masing field yang bisa diatur.

	>> QUIZ: Karena sangat pentingnya fields	tersebut untuk tuning JBoss EAP, coba jawab apa fungsi beberapa fields yang penting berikut:
	- share-prepared-statements
	- prepared-statement-cache-size
	- use-ccm
	- min-pool-size
	- max-pool-size
	- prefill (untuk pool)
	- set-tx-query-timeout
	- query-timeout
	- blocking-timeout-millis
	- idle-timeout-minutes
	- allocation-retry
	- allocation-retry-wait-millis
	
	Anda bisa lihat juga penjelasan detil mengenai konfigurasi datasource di [dokumentasi](https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/6.4/html/Administration_and_Configuration_Guide/sect-Datasource_Configuration.html)
