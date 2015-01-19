#Compiling and Installing on Red Hat Based Distros

First off I made a Tutorial for Debian based distros Specifically Ubuntu Located ["http://ubuntuforums.org/showthread.php?t=1964479"](HERE). This guide is actually pretty old and with the new updates to the servers I thought it was time to re build my Tutorial. Now i can hear you saying "But the thread title says Red Hat", and indeed you would be correct. For this particular guide i will be using a Red Hat Variation namely CentOS. Before we get started lets run through the basics:

This guide I will be assuming you have the OS pre-installed and ready for the picking.
For this tutorial i will be using 2 Virtual Machines (one for Mangosd the other for RealmD) during the course of this tutorial i will mark which steps for which machines (in the case you want to install on the SAME machine you may do so I am writing the guide in this fashion to show the separation of services)

###PREREQUISITES
As a prerequisite you should understand the following:
+ Knowledge of BASH a Plus however not required
+ Users should also be fairly familiar with the Yellowdog Updater, Modified (yum package manager)
+ Since some of the repositories in ClearOS are disabled or are specifically for ClearOS users should also be familiar with wget for other required packages
+ Users should be knowledgeable with git and BASIC functionality we will need this for cloning repos
+ usage of Putty and WinSCP is a bonus however if your doing this on the box its self you may not need these tools

###PORTS
Because this is a Red Hat derivative firewalls are more prominently installed from the get-go i will give a list of ports that you will need in order to use the system as this is an EAQ (Excessively Asked Question)
+ 22 SSH - Remote Shell (Operating System not Mangos Console)
+ 8085 mangos - client to world connection
+ 3724 realmd - client to Authentication Server
+ 7878 SOAP - (OPTIONAL) Soap interface
+ 3443 Remote Access (OPTIONAL) - Remote Console Access (using PUTTY this needs a RAW connection)
+ 80 HTTP - (OPTIONAL) if you decide to use the GM Commands Script provided or if you have a web-page you want on your server


Also for this tutorial i will be using the MASTER branch when i git clone if you would like to specifically choose the branch to clone i would suggest reading a bit of documentation about choosing branches on github before we get started.

###PREINSTALL
[code]yum groupinstall "Development Tools" "Additional Development"[/code]
The above groupinstalls might be a bit overkill for installing libraries and dependencies how ever it ensures all necessary libraries are installed with the exception of ACE
Ace was not available in the core repositories during the creation of the tutorial and should be downloaded and installed manually i will explain how in a minute.
As of the time of this writing I had to run wget and grab the ACE release directly.

ACE 6.3.1 is needed for MaNGOS builds (need information from Wiki)
For X86
http://download.opensuse.org/repositories/devel:/libraries:/ACE:/bugfixonly/RedHat_RHEL-6/i686/ace-6.3.1-16.el6.i686.rpm

For X86_64
http://download.opensuse.org/repositories/devel:/libraries:/ACE:/bugfixonly/RedHat_RHEL-6/x86_64/ace-6.3.1-16.el6.x86_64.rpm