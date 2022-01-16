#ConfigServer Security & Firewall (CSF)
#######################################

If you look at IT-related job postings anywhere, you will notice a steady demand for security pros. This does not only mean that cybersecurity is an interesting field of study, but also a very lucrative one.

With that in mind, in this article we will explain how to install and configure ConfigServer Security & Firewall (also known as CSF for short), a full-blown security suite for Linux, and share a couple of typical use cases. You will then be able to use CSF as a firewall and intrusion/login failure detection system to harden the servers you’re responsible for.

Installing and Configuring CSF in Linux
To begin, please note that Perl and libwww is a prerequisite to install CSF on any of the supported distributions (RHEL and CentOS, openSUSE, Debian, and Ubuntu). Since it should be available by default, there is no action required on your part unless one of the following steps returns a fatal error (in that case, use the package management system to install the missing dependencies).

~~~
# yum install perl-libwww-perl
~~~
~~~
# apt install libwww-perl
~~~
~~~
Step 1 – Download CSF
~~~
# cd /usr/src
# wget https://download.configserver.com/csf.tgz
or
# cd /usr/src
# git clone https://github.com/anonymansz/csf
~~~
~~~
Step 2 – Extract the CSF tarball
~~~
# tar xzf csf.tgz
# cd csf
~~~
Step 3 – Run the CSF Installation Script
This part of the process will check that all dependencies are installed, create the necessary directory structures and files for the web interface, detect currently open ports, and remind you to restart the csf and lfd daemons after you’re done with the initial configuration.
~~~
# sh install.sh
# perl /usr/local/csf/bin/csftest.pl
~~~
The expected output of the above command is as follows:
~~~
Testing ip_tables/iptable_filter...OK
Testing ipt_LOG...OK
Testing ipt_multiport/xt_multiport...OK
Testing ipt_REJECT...OK
Testing ipt_state/xt_state...OK
Testing ipt_limit/xt_limit...OK
Testing ipt_recent...OK
Testing xt_connlimit...OK
Testing ipt_owner/xt_owner...OK
Testing iptable_nat/ipt_REDIRECT...OK
Testing iptable_nat/ipt_DNAT...OK

RESULT: csf should function on this server
~~~

Step 4: Disable Firewall and Configure CSF
Disable firewalld if running and configure CSF.

# systemctl stop firewalld
# systemctl disable firewalld

Change TESTING = "1" to TESTING = "0" (otherwise, the lfd daemon will fail to start) and list allowed incoming and outgoing ports as a comma-separated list (TCP_IN and TCP_OUT, respectively) in /etc/csf/csf.conf as shown in the below output:

# Testing flag - enables a CRON job that clears iptables incase of
# configuration problems when you start csf. This should be enabled until you
# are sure that the firewall works - i.e. incase you get locked out of your
# server! Then do remember to set it to 0 and restart csf when you're sure
# everything is OK. Stopping csf will remove the line from /etc/crontab
#
# lfd will not start while this is enabled
TESTING = "0"

# Allow incoming TCP ports
TCP_IN = "20,21,22,25,53,80,110,143,443,465,587,993,995"

# Allow outgoing TCP ports
TCP_OUT = "20,21,22,25,53,80,110,113,443,587,993,995"
Once you are happy with the configuration, save the changes and return to the command line.

Step 5 – Restart and Test CSF
# systemctl restart {csf,lfd}
# systemctl enable {csf,lfd}
# systemctl is-active {csf,lfd}
# csf -v
Test Config Security Firewall
Test Config Security Firewall
At this point we are ready to start setting up firewall and intrusion detection rules as discussed next.

Setting up CSF and Intrusion Detection Rules
First off, you will want to inspect the current firewall rules as follows:

# csf -l
You can also stop them or reload them with:

# csf -f
# csf -r
respectively. Make sure to memorize these options – you will need them as you go along, particularly to check after making changes and restarting csf and lfd.

Example 1 – Allowing and Forbidding IP Addresses
To allow incoming connections from 192.168.0.10.

# csf -a 192.168.0.10
Similarly, you can deny connections originating from 192.168.0.11.

# csf -d 192.168.0.11
You can remove each of the above rules if you wish to do so.

# csf -ar 192.168.0.10
# csf -dr 192.168.0.11
CSF Allow and Deny IP Address
CSF Allow and Deny IP Address
Note how the use of -ar or -dr above removes existing allow and deny rules associated with a given IP address.

Example 2 – Limiting Incoming Connections by Source
Depending on the intended use of your server, you may want to limit incoming connections to a safe number on a port basis. To do so, open /etc/csf/csf.conf and search for CONNLIMIT. You can specify multiple port; connections pairs separated by commas. For example,

CONNLIMIT = "22;2,80;10"
will only allow 2 and 10 incoming connections from the same source to TCP ports 22 and 80, respectively.

Example 3 – Sending Alerts via Email
There are several alert types that you can choose. Look for EMAIL_ALERT settings in /etc/csf/csf.conf and make sure they are set to "1" to receive the associated alert. For example,

 
LF_SSH_EMAIL_ALERT = "1"
LF_SU_EMAIL_ALERT = "1"
will cause an alert to be sent to the address specified in LF_ALERT_TO each time someone successfully logs in via SSH or switches to another account using su command.

CSF Configuration Options and Usage
These following options are used to modify and control csf configuration. All the configuration files of csf are located under /etc/csf directory. If you modify any of the following files you will need to restart the csf daemon to take changes.

csf.conf : The main configuration file for controlling CSF.
csf.allow : The list of allowed IP’s and CIDR addresses on the firewall.
csf.deny : The list of denied IP’s and CIDR addresses on the firewall.
csf.ignore : The list of ignored IP’s and CIDR addresses on the firewall.
csf.*ignore : The list of various ignore files of users, IP’s.
Remove CSF Firewall
If you would like to remove CSF firewall completely, just run the following script located under /etc/csf/uninstall.sh directory.

# /etc/csf/uninstall.sh
The above command will erase CSF firewall completely with all the files and folders.
