# LAB0-1

Pada LAB sebelumnya, kita sudah menginstall JBoss EAP dan menjalankannya. Di sini kita akan mencoba memisahkan server binary files dengan instance files. Hal ini berguna untuk membuat server menjadi reusable, sehingga bisa membuat banyak EAP instance dengan tetap menggunakan single JBoss EAP installation.
Di akhir lab ini juga akan mencoba membuat sebuah queue destination.

Instalasi JBoss EAP
-------------------

EAP sudah terinstall pada lab sebelumnya.

Mempersiapkan instance EAP
----------------------------

1.  Buat folder untuk instance server EAP baru di /server/jboss/eap/server-standalone.
2.  Copy /server/jboss/eap/jboss-eap-6.4.0/standalone ke /server/jboss/eap/server-standalone.
3.  Create file untuk menjalankan server di direktori /server/jboss/eap/server-standalone/run.bat
    
    ```sh
    ../jboss-eap-6.4/bin/standalone.bat -b 0.0.0.0 -c standalone-full.xml -Djboss.server.base.dir=standalone -Djboss.node.name=server-standalone -Djboss.socket.binding.port-offset=0
    ```
    
4. Jalankan run.bat
5. EAP telah berjalan, silakan eksplor web console di http://localhost:9990

Membuat destination queue
--------------------------

1. Buka file /server/jboss/eap/server-standalone/standalone/configuration/standalone-full.xml
2. tambahkan bagian berikut:
  ```xml
  <subsystem xmlns="urn:jboss:domain:messaging:1.4">
    <hornetq-server>
      ...
      <jms-destinations>
        ...
        <jms-queue name="TestQueue">
          <entry name="java:/jms/queue/TestQueue"/>
        </jms-queue>
      </jms-destinations>
    </hornetq-server>
  </subsystem>
  ```
3. Restart server, ctrl-c dan kemudian jalankan lagi run.bat
4. Buka web console di http://localhost:9990
5. Buka Tab configuration
6. Buka bagian messaging -> destinations
7. Pilih default -> view
8. Pada table Queues akan terlihat queue yang baru saja kita buat
