---
layout: post
title: Install Dropbox On Ubuntu Server (10 & 11)
categories:
- 未分类
tags: []
published: true
date: '2013-10-06'
permalink: /2013/install-dropbox-on-ubuntu-server-10-11
---
<p><a href="http://ubuntuservergui.com/ubuntu-server-guide/install-dropbox-ubuntu-server">via</a></p>

<p>This post will help you install the Linux Dropbox client on your headless Ubuntu Server and link it up to your Dropbox account. Unlike the process of <a href="http://ubuntuservergui.com/ubuntu-server-guide/mount-s3-ubuntu-server">mounting an S3 bucket</a> we looked at before the Dropbox approach is a much better solution for sharing files. If you’re a daily Dropbox user you’ll quickly get hooked on the convenience of having your servers in the same file sharing loop as all your other Dropbox connected devices!</p>

<p>&nbsp;
<h4>Installing Dropbox</h4>
Start off by downloading the Linux version of Dropbox onto your server. These steps have been tested on Ubuntu Server 10.10 and 11.04.</p>

<p>Download the <em>32Bit Version</em>
<div>
<div>
<pre>wget -O dropbox.tar.gz "http://www.dropbox.com/download/?plat=lnx.x86"</pre>
</div>
</div>
&nbsp;</p>

<p>Download the <em>64Bit Version</em>
<div>
<div>
<pre>wget -O dropbox.tar.gz "http://www.dropbox.com/download/?plat=lnx.x86_64"</pre>
</div>
</div>
&nbsp;</p>

<p>If you unsure which version you need you can quickly check by running <em>uname -a</em>.
<div>
<div>
<pre>sudo uname -a</pre>
</div>
</div>
&nbsp;</p>

<p>If the uname output has an <em>i686</em> at the end you need the 32Bit version and if it has<em>x86_64</em> you want the 64Bit version.</p>

<p>When you extract the Dropbox archive it will automatically place its files in the home directory of the the user you’re logged in as under: <em>~/.dropbox</em>. You can always move these files later but its something to keep in mind.</p>

<p>Extract the Dropbox archive
<div>
<div>
<pre>tar -xzvf dropbox.tar.gz</pre>
</div>
</div>
&nbsp;
<h4>Linking Your Server to Your Dropbox Account</h4>
Before you take the next step you’ll want to make sure your LANG <a href="https://help.ubuntu.com/community/EnvironmentVariables" target="_blank">environment variable</a> is set to a value other than NULL.</p>

<p>Check the value of your <em>LANG</em> environment variable
<div>
<div>
<pre>printenv LANG     // outputs en_US.UTF-8 on my machine</pre>
</div>
</div>
&nbsp;</p>

<p>To connect the Dropbox client on your server to your Dropbox account you’ll need to copy the link it outputs into a browser window and then login to your Dropbox account.</p>

<p>Run dropboxd on your server
<div>
<div>
<pre>~/.dropbox-dist/dropboxd</pre>
</div>
</div>
&nbsp;</p>

<p>It will start outputting a link similar to this one every few seconds
<div>
<div>
<pre>This client is not linked to any account...
Please visit https://www.dropbox.com/cli_link?host_id=1c1497d78b543178b9349a7c1a8b087a&amp;cl=en_US to link this machine.</pre>
</div>
</div>
&nbsp;</p>

<p>The trick to a smooth link is to make sure you <strong>leave dropboxd running while you follow the link</strong>. You don’t need to access the link from the server you’re trying to install Dropbox on. You can copy and paste that link into a browser running on a separate machine and Dropbox will authorize the client running on your server.</p>

<p>Once it succeeds you’ll see the message <em>Client successfully linked, Welcome!</em> on your server and it will stop printing the authorization link.</p>

<p>Hit ctrl c to terminate the process
<div>
<div>
<pre>Client successfully linked, Welcome!</pre>
</div>
</div>
&nbsp;</p>

<p>Once the Dropbox client on your server is successfully linked it will automatically create a Dropbox folder under ~/Dropbox for the user you’re logged in as. All your folders will be visible under the Dropbox folder but since the Dropbox service isn’t actually running on your server yet you you won’t be able to see the files inside these folders until the client is running and has a check to synchronize.</p>

<p>You can manually start the service by running
<div>
<div>
<pre>~/.dropbox-dist/dropbox</pre>
</div>
</div>
&nbsp;</p>

<p>However, a better option for controlling the Dropbox client is to setup an Ubuntu service management script for it.
<h4>Start Dropbox Automatically On Boot</h4>
Dropbox provides a handy little service management script that makes it easy to start, stop and check the status of the Dropbox client.</p>

<p>Create a new file for the service management script
<div>
<div>
<pre>sudo vi /etc/init.d/dropbox</pre>
</div>
</div>
&nbsp;</p>

<p>Paste the following script into the new file
<pre lang="bash"><code>
#!/bin/sh
# dropbox service
# Replace with linux users you want to run Dropbox clients for
DROPBOX_USERS="user1 user2"
 
DAEMON=.dropbox-dist/dropbox
 
start() {
    echo "Starting dropbox..."
    for dbuser in $DROPBOX_USERS; do
        HOMEDIR=`getent passwd $dbuser | cut -d: -f6`
        if [ -x $HOMEDIR/$DAEMON ]; then
            HOME="$HOMEDIR" start-stop-daemon -b -o -c $dbuser -S -u $dbuser -x $HOMEDIR/$DAEMON
        fi
    done
}
 
stop() {
    echo "Stopping dropbox..."
    for dbuser in $DROPBOX_USERS; do
        HOMEDIR=`getent passwd $dbuser | cut -d: -f6`
        if [ -x $HOMEDIR/$DAEMON ]; then
            start-stop-daemon -o -c $dbuser -K -u $dbuser -x $HOMEDIR/$DAEMON
        fi
    done
}
 
status() {
    for dbuser in $DROPBOX_USERS; do
        dbpid=`pgrep -u $dbuser dropbox`
        if [ -z $dbpid ] ; then
            echo "dropboxd for USER $dbuser: not running."
        else
            echo "dropboxd for USER $dbuser: running (pid $dbpid)"
        fi
    done
}
 
case "$1" in
 
    start)
        start
        ;;
 
    stop)
        stop
        ;;
 
    restart|reload|force-reload)
        stop
        start
        ;;
 
    status)
        status
        ;;
 
    *)
        echo "Usage: /etc/init.d/dropbox {start|stop|reload|force-reload|restart|status}"
        exit 1
 
esac
exit 0
</code></pre>

<p>Control the Dropbox client like any other Ubuntu service
<div>
<div>
<pre>sudo service dropbox start|stop|reload|force-reload|restart|status</pre>
</div>
</div>
&nbsp;</p>

<p>&nbsp;</p>

<p><img title="Dropbox Delorean By Dropbox Artwork Team" alt="Dropbox Delorean By Dropbox Artwork Team" src="http://ubuntuservergui.com/wp-content/uploads/2011/05/dropbox-delorean.png" width="528" height="296" /></p>

<p>Depending upon the number of files you have on Dropbox and the speed of your internet connection it may take some time for the Dropbox client to synchronize everything.
<h4>Check Status with Dropbox CLI</h4>
Dropbox has a command line python script available separately to provide more functionality and details on the status of the Dropbox client.</p>

<p>Download the dropbox.py script and adjust the file permissions
<div>
<div>
<pre>wget -O ~/.dropbox/dropbox.py "http://www.dropbox.com/download?dl=packages/dropbox.py"
chmod 755 ~/.dropbox/dropbox.py</pre>
</div>
</div>
&nbsp;</p>

<p>You can download the script anywhere you like, I’ve included it along with the rest of the Dropbox files.</p>

<p>Now you can easily check the status of the Dropbox client
<div>
<div>
<pre>~/.dropbox/dropbox.py status
Downloading 125 files (303.9 KB/sec, 1 hr left)</pre>
</div>
</div>
&nbsp;</p>

<p>Get a full list of CLI commands
<div>
<div>
<pre>~/.dropbox/dropbox.py help</pre></div></div></p>

<p>Note: use dropbox help &lt;command&gt; to view usage for a specific command.</p>

<p> status       get current status of the dropboxd<br />
 help         provide help<br />
 puburl       get public url of a file in your dropbox<br />
 stop         stop dropboxd<br />
 running      return whether dropbox is running<br />
 start        start dropboxd<br />
 filestatus   get current sync status of one or more files<br />
 ls           list directory contents with current sync status<br />
 autostart    automatically start dropbox at login<br />
 exclude      ignores/excludes a directory from syncing


&nbsp;</p>

<p>Use the <em>exclude</em> command to keep specific files or folders from syncing to your server
<div>
<div>
<pre>~/.dropbox/dropbox.py help exclude</pre></div></div></p>

<p>dropbox exclude [list]<br />
dropbox exclude add [DIRECTORY] [DIRECTORY] ...<br />
dropbox exclude remove [DIRECTORY] [DIRECTORY] ...</p>

<p>"list" prints a list of directories currently excluded from syncing.  <br />
"add" adds one or more directories to the exclusion list, then resynchronizes Dropbox. <br />
"remove" removes one or more directories from the exclusion list, then resynchronizes Dropbox.<br />
With no arguments, executes "list". <br />
Any specified path must be within Dropbox.


&nbsp;</p>

<p>Once the Dropbox service is running and fully syncrhonized you can access all your Dropbox files and easily share files on your server with all your other Dropbox connected gadgets!</p>

<p>For more resources and troubleshooting tips visit the <a href="http://forums.dropbox.com/" target="_blank">Dropbox forums</a>. Happy syncing!</p>
