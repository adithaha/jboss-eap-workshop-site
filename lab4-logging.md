# LAB4: Logging

Default logging di JBoss EAP menggunakan periodik harian pada file server.log. Konsepnya sama seperti log4j.
Konfigurasi logging diperlukan jika kita ingin memisahkan logging pada aplikasi tertentu.

Konfigurasi untuk memisahkan log untuk package org.jboss.app.*

 1. Masuk ke http://localhost:9990
 2. Pilih tab Configuration
 3. Klik Core -> logging
 4. Terlihat root logger pada level info untuk semua package
 5. Pilih tab Handler -> Periodic
 6. Klik Add:
    Name: FILE_jboss_app
    Log Level: DEBUG
    File Path: jboss-app.log
    Suffix: .yyyy-MM-dd
    Save
 7. Pilih tab Log Categories -> Add:
    Name: org.jboss.app
    Log Level: DEBUG
    Save
 8. Pilih org.jboss.app
 9. Kolom Detail -> Handlers -> Add
 10. Pilih handler FILE_jboss_app yang tadi kita buat
 11. Selesai, berikutnya setiap ada log dari package org.jboss.app.*, akan ditulis ke jboss-app.log
