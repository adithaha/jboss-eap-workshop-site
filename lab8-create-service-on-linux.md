# Create service JBoss EAP domain mode on Linux

> Catatan: JBoss EAP yang digunakan adalah versi 6.4. 
  Apa yang ditulis disini mungkin sesuai untuk semua JBoss EAP versi > 6.0 atau JBoss AS versi 7.1 dan WildFly versi 8.X


## Create service for JBoss EAP domain master node

To enable EAP autostart on server, EAP service will be installed.

Log in as root, copy file from <eap>/bin/init.d to /etc/jboss-as and /etc/init.d

```
$ cd /etc
$ mkdir jboss-as
$ cp /apps/eap/jboss-eap-6.4/bin/init.d/jboss-as.conf /etc/jboss-as/. $ cp /apps/eap/jboss-eap-6.4/bin/init.d/jboss-as-domain.sh /etc/init.d
```

Configure jboss-as.conf

```
$ vi /etc/jboss-as/jboss.conf
...
JBOSS_USER=<user-jboss>
JBOSS_HOME=<dir-eap>
JBOSS_DOMAIN_HOME=<dir-eap-domain>
JBOSS_DOMAIN_CONFIG=domain.xml
JBOSS_HOST_CONFIG=host.xml
IP_BINDING=<this-host>
IP_BINDING_MANAGEMENT=<this-host>
...
```

Configure jboss-as-domain.sh

```
$ vi /etc/init.d/jboss-as-domain.sh 
...
if [ -z "$JBOSS_HOME" ]; then
  JBOSS_HOME=/usr/share/jboss-as
fi
export JBOSS_HOME

if [ -z "$JBOSS_DOMAIN_HOME" ]; then  <-- add this
  JBOSS_DOMAIN_HOME=$JBOSS_HOME/domain
fi

export JBOSS_DOMAIN_HOME   <-- add this

JBOSS_SCRIPT=$JBOSS_HOME/bin/domain.sh

JBOSS_MARKER_FILE=$JBOSS_DOMAIN_HOME/tmp/startup-marker   <-- add this

if [ ! -z "$JBOSS_USER" ]; then
  if [ -r /etc/rc.d/init.d/functions ]; then
    daemon --user $JBOSS_USER LAUNCH_JBOSS_IN_BACKGROUND=1 JBOSS_PIDFILE=$JBOSS_PIDFILE $JBOSS_SCRIPT --domain-config=$JBOSS_DOMAIN_CONFIG --host-config=$JBOSS_HOST_CONFIG –Djboss.domain.base.dir=$JBOSS_DOMAIN_HOME –b $IP_BINDING –bmanagement $IP_BINDING_MANAGEMENT 2>&1 > $JBOSS_CONSOLE_LOG & <-- replace this
  else
    su - $JBOSS_USER -c "LAUNCH_JBOSS_IN_BACKGROUND=1 JBOSS_PIDFILE=$JBOSS_PIDFILE $JBOSS_SCRIPT --domain-config=$JBOSS_DOMAIN_CONFIG --host-config=$JBOSS_HOST_CONFIG –Djboss.domain.base.dir=$JBOSS_DOMAIN_HOME –b $IP_BINDING –bmanagement $IP_BINDING_MANAGEMENT " 2>&1 > $JBOSS_CONSOLE_LOG & <-- replace this
  fi
fi
...
```

Add EAP as service:

```
$ chkconfig --add jboss-as-domain.sh
```

Make service start automatically when server restart:
```
$ chkconfig jboss-as-domain.sh on
```

## Create service for JBoss EAP domain slave node

To enable EAP autostart on server, EAP service will be installed.

Log in as root, copy file from <eap>/bin/init.d to /etc/jboss-as and /etc/init.d

```
$ cd /etc
$ mkdir jboss-as
$ cp /apps/eap/jboss-eap-6.4/bin/init.d/jboss-as.conf /etc/jboss-as/. $ cp /apps/eap/jboss-eap-6.4/bin/init.d/jboss-as-domain.sh /etc/init.d
```

Configure jboss-as.conf

```
$ vi /etc/jboss-as/jboss.conf
...
JBOSS_USER=<user-jboss>
JBOSS_HOME=<dir-eap>
JBOSS_DOMAIN_HOME=<dir-eap-domain>
JBOSS_DOMAIN_ADDRESS=<master-host>
JBOSS_HOST_CONFIG=host-slave.xml
IP_BINDING=<this-host>
IP_BINDING_MANAGEMENT=<this-host>
...
```

Configure jboss-as-domain.sh

```
$ vi /etc/init.d/jboss-as-domain.sh 
...
if [ -z "$JBOSS_HOME" ]; then
  JBOSS_HOME=/usr/share/jboss-as
fi
export JBOSS_HOME

if [ -z "$JBOSS_DOMAIN_HOME" ]; then  <-- add this
  JBOSS_DOMAIN_HOME=$JBOSS_HOME/domain
fi

export JBOSS_DOMAIN_HOME   <-- add this

JBOSS_SCRIPT=$JBOSS_HOME/bin/domain.sh

JBOSS_MARKER_FILE=$JBOSS_DOMAIN_HOME/tmp/startup-marker   <-- add this

if [ ! -z "$JBOSS_USER" ]; then
  if [ -r /etc/rc.d/init.d/functions ]; then
    daemon --user $JBOSS_USER LAUNCH_JBOSS_IN_BACKGROUND=1 JBOSS_PIDFILE=$JBOSS_PIDFILE $JBOSS_SCRIPT --domain-config=$JBOSS_DOMAIN_CONFIG --host-config=$JBOSS_HOST_CONFIG – Djboss.domain.master.address=$JBOSS_DOMAIN_ADDRESS – Djboss.domain.base.dir=$JBOSS_DOMAIN_HOME –b $IP_BINDING –bmanagement $IP_BINDING_MANAGEMENT 2>&1 > $JBOSS_CONSOLE_LOG & <-- replace this
  else
    su - $JBOSS_USER -c "LAUNCH_JBOSS_IN_BACKGROUND=1 JBOSS_PIDFILE=$JBOSS_PIDFILE $JBOSS_SCRIPT --domain-config=$JBOSS_DOMAIN_CONFIG --host-config=$JBOSS_HOST_CONFIG – Djboss.domain.master.address=$JBOSS_DOMAIN_ADDRESS – Djboss.domain.base.dir=$JBOSS_DOMAIN_HOME –b $IP_BINDING –bmanagement $IP_BINDING_MANAGEMENT " 2>&1 > $JBOSS_CONSOLE_LOG & <-- replace this
  fi
fi
...
```

Add EAP as service:

```
$ chkconfig --add jboss-as-domain.sh
```

Make service start automatically when server restart:
```
$ chkconfig jboss-as-domain.sh on
```
