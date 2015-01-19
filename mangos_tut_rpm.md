#Compiling and Installing on Red Hat Based Distros

First off I made a Tutorial for Debian based distros specifically Ubuntu located [here](http://ubuntuforums.org/showthread.php?t=1964479). This guide is actually pretty old and with the new updates to the servers I thought it was time to re build my Tutorial. Now I can hear you saying "But the thread title says Red Hat", and indeed you would be correct. For this particular guide I will be using a Red Hat Variation namely CentOS. Before we get started lets run through the basics:

This guide I will be assuming you have the OS pre-installed and ready for the picking.
For this tutorial I will be using 2 Virtual Machines (one for Mangosd the other for RealmD) during the course of this tutorial I will mark which steps are for which machines (in the case you want to install on the *same* machine you may do so I am writing the guide in this fashion to show the separation of services)

###PREREQUISITES
As a prerequisite you should understand the following:
+ Knowledge of BASH a Plus however not required
+ Knowledge of command line text editors (i will use nano in this guide however you are more then welcome to use vi or even vim)
+ Users should also be fairly familiar with the Yellowdog Updater, Modified (yum package manager)
+ Since some of the repositories in ClearOS are disabled or are specifically for ClearOS users should also be familiar with wget for other required packages
+ Users should be knowledgeable with git and BASIC functionality we will need this for cloning repos
+ usage of Putty and WinSCP is a bonus however if your doing this on the box its self you may not need these tools

###PORTS
Because this is a Red Hat derivative firewalls are more prominently installed from the get-go I will give a list of ports that you will need in order to use the system as this is an EAQ (Excessively Asked Question)
+ 22 SSH - Remote Shell (Operating System not Mangos Console)
+ 8085 mangos - client to world connection
+ 3724 realmd - client to Authentication Server
+ 7878 SOAP - (OPTIONAL) Soap interface
+ 3443 Remote Access (OPTIONAL) - Remote Console Access (using PUTTY this needs a RAW connection)
+ 80 HTTP - (OPTIONAL) if you decide to use the GM Commands Script provided or if you have a web-page you want on your server
+ 3306 MySQL - (OPTIONAL) if you want to manage your MySQL or MariaDB with a remote host or you are setting up 2 servers

###PRE-INSTALLATION NOTES
Some things to keep in mind as we walk through the installation:
+ CentOS 7 X64
+ I will be using the MASTER branch when I git clone if you would like to specifically choose the branch to clone I would suggest reading a bit of documentation about choosing branches on github before we get started.
+ I will be using MariaDB 10.0 as a replacement for MySQL


###PREINSTALL
Set up a YUM Repo to use the MariaDB official repositories
```bash
nano /etc/yum.repos.d/MariaDB.repo
```
now in our newly created repo file we will set the following options
```bash
# MariaDB 10.0 CentOS repository list - created 2015-01-19 02:15 UTC
# http://mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.0/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```
After the file is in place, install MariaDB with:
```bash
yum install MariaDB-server MariaDB-client
```
you will be prompted to accept the GPG key after the packages have been downloaded. Once all of that is installed we can move on to the rest of the development tool and libraries with the command:
```bash
yum groupinstall "Development Tools" "Additional Development"
```
The above group installs might be a bit overkill for installing libraries and dependencies how ever it ensures all necessary libraries are installed with the exception of ACE.
Ace was not available in the core repositories during the creation of this tutorial and should be downloaded and installed manually (in the event that some one finds a repository that includes ACE I will update this guide with the new information) I will explain how in a minute.
As of the time of this writing I had to run wget and grab the ACE release directly.

ACE 6.3.1 is needed for MaNGOS builds (need information from Wiki)
```bash
## X86
wget http://download.opensuse.org/repositories/devel:/libraries:/ACE:/bugfixonly/RedHat_RHEL-6/i686/ace-6.3.1-16.el6.i686.rpm
#
## OR
## X64
wget http://download.opensuse.org/repositories/devel:/libraries:/ACE:/bugfixonly/RedHat_RHEL-6/x86_64/ace-6.3.1-16.el6.x86_64.rpm
```
once your ACE package is done downloading you may proceed to install it with:
```bash
## X86
yum localinstall ace-6.3.1-16.el6.i686.rpm
#
## OR
# X64
yum localinstall ace-6.3.1-16.el6.x86_64.rpm
```
