Simple init.d script used on RHEL 5/6 for the apache_exporter used in conjunction with Prometheus.

```
#!/bin/sh
### BEGIN INIT INFO
# Provides:          apache exporter
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Description:       apache_exporter for prometheus written in Go
### END INIT INFO

# Source function library.
. /etc/rc.d/init.d/functions


DAEMON=/usr/bin/apache_exporter
NAME=apache_exporter
USER=tomcat
PIDFILE=/var/run/prometheus/$NAME.pid
LOGFILE=/var/log/prometheus/$NAME.log

ARGS=""
[ -r /etc/default/$NAME ] && . /etc/default/$NAME

do_start_prepare()
{
    mkdir -p `dirname $PIDFILE` || true
    mkdir -p `dirname $LOGFILE` || true
    chown -R $USER: `dirname $LOGFILE`
    chown -R $USER: `dirname $PIDFILE`
}

do_start_cmd()
{
    do_start_prepare
    echo -n "Starting daemon: "$NAME
        daemon --user $USER --pidfile="$PIDFILE" "$DAEMON $ARGS >> $LOGFILE 2>&1 &"
        echo "."
}

do_stop_cmd()
{
    echo -n "Shutting down $NAME: "
    killproc $NAME
    rm -f $PIDFILE
    echo
}

case "$1" in
  start)
    do_start_cmd
    ;;
  stop)
    do_stop_cmd
    ;;
  restart)
    do_stop_cmd
    do_start_cmd
    ;;
  *)
    echo "Usage: $1 {start|stop|status|restart|uninstall}"
    exit 1
esac

exit 0

```
The above script will read any run-time arguments associated with the apache_exporter application from the /etc/default/apache_exporter file.  A sample of possible arguments is as follows:

```
ARGS="-insecure -scrape_uri https://localhost:443/server-status?auto"
```
