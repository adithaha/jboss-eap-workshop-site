# Monitoring JBoss EAP

> Catatan: JBoss EAP yang digunakan adalah versi 6.3. 
  Apa yang ditulis disini mungkin sesuai untuk semua JBoss EAP versi > 6.0 atau JBoss AS versi 7.1 dan WildFly versi 8.X

Untuk memonitor JBoss EAP kita dapat menggunakan beberapa tools diantaranya:
	
- JBoss Command Line Interface (CLI), yaitu dengan perintah `jboss-cli.sh` (Linux) atau `jboss-cli.bat` (Windows)
- Native Management API, yaitu dengan pemrograman menggunakan JBoss remoting
- HTTP Management API (REST)
- JMX Tool, seperti JConsole dan lainnya.
- JBoss Operation Network atau RHQ

# Monitoring Datasource atau Connection Pool

Pada LAB ini kita akan mencoba melakukan pengaturan mode monitoring untuk data source di JBoss EAP, kemudian memonitor menggunakan CLI, HTTP management API, JConsole serta membuat tool sederhana untuk monitoring secara grafis.

## Enable Datasource Statistic 

Untuk memonitor datasource atau koneksi ke database, kita perlu meng-enable statistic dengan cara menambahkan konfigurasi `statistics-enabled="true"` pada elemen datasource yang ingin kita monitor, seperti dibawah ini:


```xml
<datasource jndi-name="..." pool-name="..." enabled="true" use-java-context="true" statistics-enabled="true">
```

Atau bisa juga dengan menggunakan JBoss CLI, untuk mode standalone:

```
[standalone@localhost:9999 /] /subsystem=datasources/data-source=ExampleDS:write-attribute(name=statistics-enabled,value=true)
{
    "outcome" => "success",
    "response-headers" => {
        "operation-requires-reload" => true,
        "process-state" => "reload-required"
    }
}
```

Pada mode domain:

```
[domain@localhost:9999 /] /profile=full/subsystem=datasources/data-source=ExampleDS:write-attribute(name=statistics-enabled,value=true)
{
    "outcome" => "success",
    "result" => undefined,
    "server-groups" => {"main-server-group" => {"host" => {"master" => {
        "server-one" => {"response" => {
            "outcome" => "success",
            "response-headers" => {
                "operation-requires-restart" => true,
                "process-state" => "restart-required"
            }
        }},
        "server-two" => {"response" => {
            "outcome" => "success",
            "response-headers" => {
                "operation-requires-restart" => true,
                "process-state" => "restart-required"
            }
        }}
    }}}}
```

## Datasource & JDBC statistic

Penjelasan mengenai arti dari parameter (metric) yang ditampilkan pada statistic liat di [Datasource Statistics](https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/6.3/html/Administration_and_Configuration_Guide/Datasource_Statistics.html)

## Memonitor menggunakan JBoss CLI

Berikut cara memonitor menggunakan CLI 

```
/subsystem=datasources/data-source=ExampleDS/statistics=pool:read-resource(recursive=true, include-runtime=true)
```

Untuk JBoss EAP yang jalan dalam mode domain, perintah diatas menjadi seperti ini:

```
/host=master/server=server-one/subsystem=datasources/data-source=ExampleDS/statistics=pool:read-resource(include-runtime=true)
```

## Memonitor dari HTTP Management API

JBoss EAP memiliki management interface yang jalan 

`http://localhost:9990/management/subsystem/datasources/data-source/ExampleDS/statistics/pool?include-runtime=true`

Berikut hasil output dari akses URL tersebut menggunakan browser:

```
{
"ActiveCount": "7",
"AvailableCount": "4",
"AverageBlockingTime": "1",
"AverageCreationTime": "37",
"CreatedCount": "7",
"DestroyedCount": "0",
"InUseCount": "6",
"MaxCreationTime": "253",
"MaxUsedCount": "7",
"MaxWaitCount": "0",
"MaxWaitTime": "1",
"TimedOut": "0",
"TotalBlockingTime": "1",
"TotalCreationTime": "259",
"statistics-enabled": true
}
```

## Memonitor dengan menggunakan JConsole

JConsole adalah JMX graphical-tool bawaan dari distribusi JRE/JDK untuk memonitor JVM atau aplikasi yang mendukung JMX.

Berikut langkah-langkah untuk memonitor menggunakan JConsole:

1. Jalankan JConsole GUI dengan menggunakan perintah `jconsole.sh` (Linux) atau `jconsole.bat` (Windows) yang ada di direktori `<JBOSS_EAP_HOME>/bin/`

2. Pilih __Remote Process__ dan masukan URL `service:jmx:remoting-jmx://HOSTNAME:PORT`, dengan `HOSTNAME` adalah hostname atau IP address dari JBoss EAP server dan PORT adalah port management (native) yang bisa dilihat di file konfigurasi (standalone.xml) seperti dibawah ini:

	```
	<socket-binding name="management-native" interface="management" port="${jboss.management.native.port:9999}"/>
	```

	Jika port-offset tidak diset, berarti port yang harus kita gunakan adalah port 9999.
	
	![image](https://cloud.githubusercontent.com/assets/3068071/7292291/dceccc0c-e9c4-11e4-98bb-391df91a4f70.png)
	
	>> Kita juga dapat menggunakan __Local Process__ jika JConsole yang kita jalankan kita gunakan untuk memonitor atau mengeloala JBoss EAP yang jalan di mesin/host yang sama. Pilih  pada __Local Process__ list suatu proses dengan nama seperti ini `jboss-module.jar -mp /path/...` 

3. Set username/password, lalu klik tombol "Connect"




