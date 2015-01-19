#Compiling and Installing on Red Hat Based Distros

First off I made a Tutorial for Debian based distros specifically Ubuntu located [here](http://ubuntuforums.org/showthread.php?t=1964479). This guide is actually pretty old and with the new updates to the servers I thought it was tI'me to re build my Tutorial. Now I can hear you saying "But the thread title says Red Hat", and indeed you would be correct. For this particular guide I will be using a Red Hat Variation namely CentOS. Before we get started lets run through the basics:

This guide I will be assuming you have the OS pre-installed and ready for the picking.
For this tutorial I will be using 2 Virtual Machines (one for Mangosd the other for RealmD) during the course of this tutorial I will mark which steps are for which machines (in the case you want to install on the *same* machine you may do so I am writing the guide in this fashion to show the separation of services)

###PREREQUISITES
As a prerequisite you should understand the following:
+ Knowledge of BASH a Plus however not required
+ Knowledge of command line text editors (i will use nano in this guide however you are more then welcome to use vi or even vI'm) Debian users that are used to "pico" should probably use nano as its praticly the same
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
+ Default user on CentOS 7 is root, you may create a new user with out root privleges however for all the installs you will need root privleges. As i will be using root i will mark the areas you need to run commands with sudo for clarification.
+ I will be using the MASTER branch when I git clone if you would like to specifically choose the branch to clone I would suggest reading a bit of documentation about choosing branches on github before we get started.
+ I will be using MariaDB 10.0 as a replacement for MySQL


###PREINSTALL
Set up a YUM Repo to use the MariaDB official repositories
```bash
# sudo if not root
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
# sudo if not root
yum install MariaDB-server MariaDB-client
```
Depending on your internet speed this might be a good time to get a cup of coffee during the creation of this tutorial I had about 7 hundred packages to install as well as updates.
you will be prompted to accept the GPG key after the packages have been downloaded. Once all of that is installed we can move on to the rest of the development tools and libraries with the command:
```bash
# sudo if not root
yum groupinstall "Development Tools" "Additional Development"
```
The above group installs might be a bit overkill for installing libraries and dependencies how ever it ensures all necessary libraries are installed with the exception of ACE.
Ace was not available in the core repositories during the creation of this tutorial and should be downloaded and installed manually (in the event that some one finds a repository that includes ACE I will update this guide with the new information) I will explain how in a minute.
As of the time of this writing I had to run wget and grab the ACE release directly.

ACE 6.3.1 is needed for newer MaNGOS builds
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
# sudo if not root
## X86
yum localinstall ace-6.3.1-16.el6.i686.rpm
#
## OR
# X64
yum localinstall ace-6.3.1-16.el6.x86_64.rpm
```
at this point you should have a clean working environment ready to go. and now we can start pull down the MaNGOS sources and set up our build environment.
###BUILDING MANGOS
 At this time lets create our base directory tree from my home directory I'm going to create a SOURCES directory in which i will run all my git clones it is in this base directory where i keep all of my code.
 I'm going to create all of these directories in order to keep all of my repositories organized in a somewhat orderly fashion.
 ```bash
 mkdir SOURCES
 cd SOURCES
 git clone https://github.com/mangosthree/server.git
 git clone https://github.com/mangosthree/database.git
 ```
 Now you should have bolth the database and the server code ready to go however we need to add the scripts library in order to have that compile with our our core
```bash
git clone https://github.com/mangosthree/scripts.git server/src/bindings/scripts
```
now you will note i am placing the scripts into the `server/src/bindings/scripts` directory this is rather important as we will be setting this as on of our compile options later on. Its important to note here that this is a step that can be hard to understand for a first timer but we wont get into depth in this tutorial about it.
once all of our downloads are done we can create a few more directories and kick off the compile.
```bash
mkdir _build && cd _build
```
and now for the huge command of the day.
```bash
cmake .. -DTBB_USE_EXTERNAL=1 -DCMAKE_BUILD_TYPE=release -DACE_USE_EXTERNAL=1 -DCMAKE_INSTALL_PREFIX=/opt/mangos -DINCLUDE_BINDINGS_DIR=scripts -DPCH=0 -DCMAKE_CXX_FLAGS="-O3 -march=native" -DCMAKE_C_FLAGS="-O3 -march=native"
```
at this point if you get something like this:
```bash
CMake Error at CMakeLists.txt:137 (message):
  This project requires ACE installed when ACE_USE_EXTERNAL is set.  Please
  download the ACE Micro Release Kit from http://download.dre.vanderbilt.edu/
  and install it.  If this script didn't find ACE and it was correctly
  installed please set ACE_ROOT to the correct path.
```
then ACE hasnt been correctly installed If you ran the above install (yum localinstall ace) you need to ALSO set the ACE_ROOT so that cmake can acuratly determine where in your path its located
now at first this one line looks pretty scary, but lets take a closer look and break it down to understand what were doing here
+ -DTBB_USE_EXTERNAL=1 this will instruct the compiler to locate and use the External TBB installation (should have been installed with the "Additional Development" group)
+ -DCMAKE_BUILD_TYPE=release for debugging purposes you may change this to debug
+ -DACE_USE_EXTERNAL=1 remember that ACE package we downloaded earlier? Yes this is where the compiler is told to use that
+ -DCMAKE_INSTALL_PREFIX=/opt/mangos this is our installation directory feel free to change this to any location you want however i prefer to have it here as this is the "optional software directory"
+ -DINCLUDE_BINDINGS_DIR=scripts this is the scripts directory that we cloned before you must have the cloned directory inside of the src/bindings directory
+ -DPCH=0 Pre-compiled headers option (tends to speed compile up however i dont think pre-compiled headers ship with mangos if any one would like to clarify on that id be happy to add the information)
I found these next two off another wiki page for architecture optimization options essentially it stream-lines the code for your hardware
+ -DCMAKE_CXX_FLAGS="-O3 -march=native"
+ -DCMAKE_C_FLAGS="-O3 -march=native"
