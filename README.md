Hello everyone, I am Gaurav with VOIP development experience. Today we will be learning "How to Install a Freeswitch on Linux system and configuring freeswitch default users." Its very easy just follow the below steps. 

*************************************************************************
INSTALLING FREESWITCH
*************************************************************************
1. Add repository
Add freeswitch repository
--------------------------
# curl http://files.freeswitch.org/repo/deb/debian/freeswitch_archive_g0.pub | sudo apt-key add -
# echo "deb http://files.freeswitch.org/repo/deb/freeswitch-1.6/ jessie main" | sudo tee /etc/apt/sources.list.d/freeswitch.list
# sudo apt-get update

2. Install dependencies
Install all the dependencies needed by freeswitch
--------------------------------------------------
# sudo apt-get install -y libyuv-dev libvpx2-dev liblua5.2-dev libvpx2-dev libvpx2 zlib1g-dev libspeex1 libopus-dev libsndfile-dev autoconf automake devscripts gawk g++ git-core 'libjpeg-dev|libjpeg62-turbo-dev' libncurses5-dev 'libtool-bin|libtool' make python-dev gawk pkg-config libtiff5-dev libperl-dev libgdbm-dev libdb-dev gettext libssl-dev libcurl4-openssl-dev libpcre3-dev libspeex-dev libspeexdsp-dev libsqlite3-dev libedit-dev libldns-dev libpq-dev yasm

3. Download freeswitch
Create the source folder of your choice and download freeswitch code (shared above) in this location
-----------------------------------------------------------------------------------------
# mkdir /root/sources && mkdir /usr/local/switch && cd /root/sources


4. Compile/Install freeswitch
To compile freeswitch, navigate to the freeswitch source directory.
-------------------------------------------------------------------------
# ./bootstrap.sh -j
# ./configure --enable-core-pgsql-support --prefix="/usr/local/switch/" --with-java=/usr/lib/jvm/java-8-oracle/include/

5. Installing Freeswitch
 Install freeswitch using following command
-------------------------------------------------
# make && make install

6. Install few sound files
Install in /usr/local/switch/share/freeswitch/sounds/
---------------------------------------------------------
# make cd-sounds-install cd-moh-install

7. Starting freeswitch with following command
---------------------------------------------------
# cd /usr/local/switch/bin
#./freeswitch

8. Add freeswitch user to your system, and change the required permissions for the directory.
--------------------------------------------------------------------------------------------------------
# adduser --disabled-password  --quiet --system --home /usr/local/switch --gecos "FreeSWITCH Voice Platform" --ingroup daemon freeswitch
# chown -R freeswitch:daemon /usr/local/switch/ 
# chmod -R o-rwx /usr/local/switch/

9. Create the file /etc/init.d/freeswitch with the following code, but before that create the location for PID file and grant permission for freeswitch
---------------------------------------------------------------------------------------------------------------------------------------------------
# mkdir /usr/local/switch/run
# chown -R freeswitch:daemon /usr/local/switch/run
# vi /etc/init.d/freeswitch

10. Using your favorite editor add the following in /etc/init.d/freeswitch  and save the file.
-------------------------------------------------------------------------------------------------
#!/bin/bash
### BEGIN INIT INFO
# Provides:          freeswitch
# Required-Start:    $local_fs $remote_fs
# Required-Stop:     $local_fs $remote_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Description:       Freeswitch debian init script.
#
### END INIT INFO
# Do NOT "set -e"
# PATH should only include /usr/* if it runs after the mountnfs.sh script
PATH=/sbin:/usr/sbin:/bin:/usr/bin:/usr/local/bin
DESC="Freeswitch"
NAME=freeswitch
DAEMON=/usr/local/switch/bin/$NAME
DAEMON_ARGS="-nc"
PIDFILE=/usr/local/switch/var/run/freeswitch/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
FS_USER=freeswitch
FS_GROUP=daemon
# Exit if the package is not installed
[ -x "$DAEMON" ] || exit 0
# Read configuration variable file if it is present
[ -r /etc/default/$NAME ] && . /etc/default/$NAME
# Load the VERBOSE setting and other rcS variables
. /lib/init/vars.sh
# Define LSB log_* functions.
# Depend on lsb-base (>= 3.0-6) to ensure that this file is present.
. /lib/lsb/init-functions
#
# Function that sets ulimit values for the daemon
#
do_setlimits() {
        ulimit -c unlimited
        ulimit -d unlimited
        ulimit -f unlimited
        ulimit -i unlimited
        ulimit -n 999999
        ulimit -q unlimited
        ulimit -u unlimited
        ulimit -v unlimited
        ulimit -x unlimited
        ulimit -s 240
        ulimit -l unlimited
        return 0
}
#
# Function that starts the daemon/service
#
do_start()
{
    # Set user to run as
        if [ $FS_USER ] ; then
      DAEMON_ARGS="`echo $DAEMON_ARGS` -u $FS_USER"
        fi
    # Set group to run as
        if [ $FS_GROUP ] ; then
          DAEMON_ARGS="`echo $DAEMON_ARGS` -g $FS_GROUP"
        fi
        # Return
        #   0 if daemon has been started
        #   1 if daemon was already running
        #   2 if daemon could not be started
        start-stop-daemon --start --quiet --pidfile $PIDFILE --exec $DAEMON --test > /dev/null -- \
                || return 1
        do_setlimits
        start-stop-daemon --start --quiet --pidfile $PIDFILE --exec $DAEMON --background -- \
                || return 2
        # Add code here, if necessary, that waits for the process to be ready
        # to handle requests from services started subsequently which depend
        # on this one.  As a last resort, sleep for some time.
}
#
# Function that stops the daemon/service
#
do_stop()
{
        # Return
        #   0 if daemon has been stopped
        #   1 if daemon was already stopped
        #   2 if daemon could not be stopped
        #   other if a failure occurred
        start-stop-daemon --stop --quiet --retry=TERM/30/KILL/5 --pidfile $PIDFILE --name $NAME
        RETVAL="$?"
        [ "$RETVAL" = 2 ] && return 2
        # Wait for children to finish too if this is a daemon that forks
        # and if the daemon is only ever run from this initscript.
        # If the above conditions are not satisfied then add some other code
        # that waits for the process to drop all resources that could be
        # needed by services started subsequently.  A last resort is to
        # sleep for some time.
        start-stop-daemon --stop --quiet --oknodo --retry=0/30/KILL/5 --exec $DAEMON
        [ "$?" = 2 ] && return 2
        # Many daemons don't delete their pidfiles when they exit.
        rm -f $PIDFILE
        return "$RETVAL"
}
#
# Function that sends a SIGHUP to the daemon/service
#
do_reload() {
        #
        # If the daemon can reload its configuration without
        # restarting (for example, when it is sent a SIGHUP),
        # then implement that here.
        #
        start-stop-daemon --stop --signal 1 --quiet --pidfile $PIDFILE --name $NAME
        return 0
}
case "$1" in
  start)
        [ "$VERBOSE" != no ] && log_daemon_msg "Starting $DESC" "$NAME"
        do_start
        case "$?" in
                0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
                2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
        esac
        ;;
  stop)
        [ "$VERBOSE" != no ] && log_daemon_msg "Stopping $DESC" "$NAME"
        do_stop
        case "$?" in
                0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
                2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
        esac
        ;;
  status)
       status_of_proc -p $PIDFILE $DAEMON $NAME && exit 0 || exit $?
       ;;
  #reload|force-reload)
        #
        # If do_reload() is not implemented then leave this commented out
        # and leave 'force-reload' as an alias for 'restart'.
        #
        #log_daemon_msg "Reloading $DESC" "$NAME"
        #do_reload
        #log_end_msg $?
        #;;
  restart|force-reload)
        #
        # If the "reload" option is implemented then remove the
        # 'force-reload' alias
        #
        log_daemon_msg "Restarting $DESC" "$NAME"
        do_stop
        case "$?" in
          0|1)
                do_start
                case "$?" in
                        0) log_end_msg 0 ;;
                        1) log_end_msg 1 ;; # Old process is still running
                        *) log_end_msg 1 ;; # Failed to start
                esac
                ;;
          *)
                # Failed to stop
                log_end_msg 1
                ;;
        esac
        ;;
  *)
        #echo "Usage: $SCRIPTNAME {start|stop|restart|reload|force-reload}" >&2
        echo "Usage: $SCRIPTNAME {start|stop|restart|force-reload}" >&2
        exit 3
        ;;
esac
exit 0

11. Make the script executable and make it auto start on system boot
---------------------------------------------------------------------------
# chmod +x /etc/init.d/freeswitch
# update-rc.d freeswitch defaults

Freeswitch is now installed in your system. Wasn't it easy ??

12. Exiting freeswitch
-----------------------
# /quit

*************************************************************************
ADDING FREESWITCH USERS
*************************************************************************
1. To add a freeswitch user, navigate to /usr/local/switch/etc/freeswitch/directory/default directory and create a file by the name 1000.xml and add the following.
# cd /usr/local/switch/etc/freeswitch/directory/default
# vi 1000.xml

<include>
<user id=”1000?>
<params>
<param name=”password” value=”default.1000?/>
<param name=”vm-password” value=”1000?/>
</params>
<variables>
<variable name=”toll_allow” value=”domestic,international,local”/>
<variable name=”accountcode” value=”1000?/>
<variable name=”user_context” value=”default”/>
<variable name=”effective_caller_id_name” value=”Extension 1000?/>
<variable name=”effective_caller_id_number” value=”1000?/>
<variable name=”outbound_caller_id_name” value=”$${outbound_caller_name}”/>
<variable name=”outbound_caller_id_number” value=”$${outbound_caller_id}”/>
<variable name=”callgroup” value=”techsupport”/>
</variables>
</user>
</include>

2. Change the ownership.
# chown freeswitch:daemon 1000.xml

3. Similarly, create another user in 1001.xml
<include>
<user id=”1001?>
<params>
<param name=”password” value=”default.1001?/>
<param name=”vm-password” value=”1001?/>
</params>
<variables>
<variable name=”toll_allow” value=”domestic,international,local”/>
<variable name=”accountcode” value=”1001?/>
<variable name=”user_context” value=”default”/>
<variable name=”effective_caller_id_name” value=”Extension 1001?/>
<variable name=”effective_caller_id_number” value=”1001?/>
<variable name=”outbound_caller_id_name” value=”$${outbound_caller_name}”/>
<variable name=”outbound_caller_id_number” value=”$${outbound_caller_id}”/>
<variable name=”callgroup” value=”techsupport”/>
</variables>
</user>
</include>

4. Change the ownership.
# chown freeswitch:daemon 1001.xml


Finally you can test this solution by using any SIP phones (like : Zoiper/SJ Phone/pjsip/xlite/linphone, etc).
Below are the details to register for the installed Freeswitch

User     : 1000
Password : default.1000
Domain   : demohost.hostingwikipedia.com

User     : 1001
Password : default.1001
Domain   : demohost.hostingwikipedia.com
