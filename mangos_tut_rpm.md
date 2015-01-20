#Compiling and Installing on Red Hat Based Distros

First off I made a Tutorial for Debian based distros specifically Ubuntu located [here](http://ubuntuforums.org/showthread.php?t=1964479). This guide is actually pretty old and with the new updates to the servers I thought it was tI'me to re build my Tutorial. Now I can hear you saying "But the thread title says Red Hat", and indeed you would be correct. For this particular guide I will be using a Red Hat Variation namely CentOS. Before we get started lets run through the basics:

This guide I will be assuming you have the OS pre-installed and ready for the picking.
For this tutorial I will be using 2 Virtual Machines (one for Mangosd the other for RealmD) during the course of this tutorial I will mark which steps are for which machines (in the case you want to install on the *same* machine you may do so I am writing the guide in this fashion to show the separation of services)

###PREREQUISITES
As a prerequisite you should understand the following:
+ Knowledge of BASH a Plus however not required
+ Knowledge of command line text editors (i will use nano in this guide however you are more then welcome to use vi or even vim) Debian users that are used to "pico" should probably use nano as its praticly the same
+ Users should also be fairly familiar with the Yellowdog Updater, Modified (yum package manager)
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
once we add the MariaDB repo lets also add the Repository for the ACE installs
```bash
nano /etc/yum.repos.d/ace_micro.repo
```
and in this file we will add the following:
```bash
[devel_libraries_ACE_micro]
name=Latest ACE micro release (CentOS_7)
type=rpm-md
baseurl=http://download.opensuse.org/repositories/devel:/libraries:/ACE:/micro/CentOS_7/
gpgcheck=1
gpgkey=http://download.opensuse.org/repositories/devel:/libraries:/ACE:/micro/CentOS_7/repodata/repomd.xml.key
enabled=1
```
After the file is in place, install MariaDB and ACE with:
```bash
# sudo if not root
yum install MariaDB-server MariaDB-client ace-6.3.1
```
Depending on your internet speed this might be a good time to get a cup of coffee during the creation of this tutorial I had about 7 hundred packages to install as well as updates.
you will be prompted to accept the GPG key after the packages have been downloaded. Once all of that is installed we can move on to the rest of the development tools and libraries with the command:
```bash
# sudo if not root
yum groupinstall "Development Tools" "Additional Development"
```
The above group installs might be a bit overkill for installing libraries and dependencies how ever it ensures all necessary libraries are installed with the exception of ACE.
at this point you should have a clean working environment ready to go. and now we can start to pull down the MaNGOS sources and set up our build environment.
###BUILDING MANGOS
At this time lets create our base directory tree from my home directory I'm going to create a SOURCES directory in which i will run all my git clones it is in this base directory where i keep all of my code.
I'm going to create all of these directories in order to keep all of my repositories organized in a somewhat orderly fashion.
```bash
mkdir SOURCES
cd SOURCES
git clone https://github.com/mangosthree/server.git
git clone https://github.com/mangosthree/database.git
```
Now you should have both the database and the server code ready to go however we need to add the scripts library in order to have that compile with our our core
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
cmake .. -DTBB_USE_EXTERNAL=1 -DCMAKE_BUILD_TYPE=release -DACE_USE_EXTERNAL=0 -DCMAKE_INSTALL_PREFIX=/opt/mangos -DINCLUDE_BINDINGS_DIR=scripts -DPCH=0 -DCMAKE_CXX_FLAGS="-O3 -march=native" -DCMAKE_C_FLAGS="-O3 -march=native"
```
Now at first this one line looks pretty scary, but lets take a closer look and break it down to understand what were doing here
+ -DTBB_USE_EXTERNAL=1 this will instruct the compiler to locate and use the External TBB sources (should have been installed with the "Additional Development" group)
+ -DCMAKE_BUILD_TYPE=release for debugging purposes you may change this to debug
+ -DACE_USE_EXTERNAL=0 remember that ACE package we installed earlier? Yes this is where the compiler is told to use that If we wanted to install MaNGOS with an EXTERNAL ace package we would need to download the source tar-ball and compile ACE along side of mangos
+ -DCMAKE_INSTALL_PREFIX=/opt/mangos this is our installation directory feel free to change this to any location you want however i prefer to have it here as this is the "optional software directory"
+ -DINCLUDE_BINDINGS_DIR=scripts this is the scripts directory that we cloned before you must have the cloned directory inside of the src/bindings directory
+ -DPCH=0 Pre-compiled headers option (tends to speed compile up however i dont think pre-compiled headers ship with mangos if any one would like to clarify on that id be happy to add the information)

I found these next two off another wiki page for architecture optimization options essentially it stream-lines the code for your hardware
+ -DCMAKE_CXX_FLAGS="-O3 -march=native"
+ -DCMAKE_C_FLAGS="-O3 -march=native"

Now you should have a fully configured source directory and some output should appear on your terminal. as an example this is what mine looked like:
```bash
-- Detected 64-bit platform.
-- Using mysql-config: /usr/bin/mysql_config
-- Found MySQL library: /usr/lib64/libmysqlclient_r.so
-- Found MySQL headers: /usr/include/mysql
-- Found OpenSSL: /usr/lib64/libssl.so;/usr/lib64/libcrypto.so (found version "1.0.1e")
-- Found ZLIB: /usr/lib64/libz.so (found version "1.2.7")
-- MaNGOS-Core revision  : a681260670a3afee4f7e986a4de6dcbb7308ec81
-- Install server to     : /opt/mangos
-- Build script library  : Yes (using scripts)
-- Use PCH               : No
-- Using Linux port
-- Configuring done
-- Generating done
-- Build files have been written to: /root/SOURCES/server/_build
```
at this point we can safely run our make and then proceed to install our packages
```bash
make -j`getconf _NPROCESSORS_ONLN`
```
The option `getconf _NPROCESSORS_ONLN` instructs the server to get the number of online CPUs and returns that value to the compiler (make). Some system comparisons:
+ Dell Poweredge R900 4 quad-cores (total of 16 cores at 1.6GHz) and the server compiled in just about 2 minutes. 
+ Dell Poweredge 2850 2 dual cores (total of 4 cores at 2.8GHz) compiled in roughly 30 minutes.

so depending on your system speed (and number of cores) this step could very well take a while to complete. Once the compile is done and you dont have any errors you can proceed to install it with
```bash
# sudo if not root
make install
```
This will install your server software in the `-DCMAKE_INSTALL_PREFIX` location that we specified during the cmake configuration step. Before we go on lets look at the directory structure
+ /opt/mangos - this is our root directory (if yours is different you can substitute this value for your location)
+ /opt/mangos/etc - will be all our configuration files
+ /opt/mangos/bin - binary directory containing the realmd and the mangosd programs

##DATABSE INSTALLATION
Lets get back to our sources directory and start with adding in the required databases. Up until just recently adding all the databases and there patches was very difficult and i had created a few scripts to manage that task however i will not be using them as Factionwars, Nemok, BrainDedd and Antz have done a great job on a bash script that does most of the heavy lifting for you.
We are going to run a few commands to perform the following tasks:
+ Set up root password (in the event it wasn't done during the yum install step)
+ import realmd.sql
+ import mangos.sql
+ import characters.sql
+ import scriptdev2 database (is this needed? Ive heard some things around the wiki about moving all that into the core when will that take effect?)

now that we know what we are doing lets how we are doing it:
+ MySQL root password:
```bash
# In the even that you system didnt set a password at the time of install you will need to set the password here in order to use the installer provided
# script will defualt to "root" as the password and will not accept a blank password
mysql --user=root --password= --host=localhost
# you should be looking at the MariaDB prompt 
# MariaDB [(none)]>
```
now at this prompt we will issue a few commands in order to set our root password and secure the server.
```sql
UPDATE mysql.user SET Password = PASSWORD('new_password_here') WHERE User = 'root';
FLUSH PRIVILEGES;
exit;
```
This will set the password of 'new_password_here' to the user root, it will effect every instance of the user root so all passwords for root are now 'new_password_here'. The FLUSH statement causes the server to reread the grant tables. Without it, the password change remains unnoticed by the server until you restart it.

```bash
# /root/SOURCES is the location of all of my git cloned repositories
cd /root/SOURCES/server/sql/
mysql --user=root --host=localhost --password=pass < create_mysql.sql
cd /root/SOURCES/server/src/bindings/scripts/sql
# this should set up all the required scripts
mysql --user=root --host=localhost --password=pass < scriptdev2_create_database.sql
mysql --user=root --host=localhost --password=pass scriptdev2 < scriptdev2_create_structure_mysql.sql
mysql --user=root --host=localhost --password=pass scriptdev2 < scriptdev2_script_full.sql
cd /root/SOURCES/database/
```
At this point we can run the Linux Installer provided in the repository
```bash
bash ./install_linux.sh
```
Follow the on screen instructions. You will eventually come to a part of the script
```bash
Please enter the path to your Mangos Three repository:
```
At this point you will need to enter the path to the directory of your server code for example mine is:
```bash
/root/SOURCES/server
```
That completes the database installation

###CONFIGURATION SETTINGS
Well if your still with me this far I think your doing pretty good. We will now set our configuration settings and even set up the AH-Bot.First off lets start with the realmd section as this is the easiest to get on-line with out any trouble.

#####NOTES
+ /opt/mangos/logs - we will create this directory for our log files
+ /opt/mangos/data - we will create this directory for our data files

I haven't used to the client data extractors in quite some time ive been using a back up of my data for about a year and ive just been moving it around to different servers and new compiles, however i dont think the processes has changed all that much. If you need get your data extracting I would go a head and do that now so you can work on your configuration settings.

Back to our shell (or putty through SSH):
```bash
cd /opt/mangos
mkdir /opt/mangos/logs
mkdir /opt/mangos/data
cd /opt/mangos/etc
```
now inside of the this directory you should see 3 files
+ mangosd.conf.dist
+ realmd.conf.dist
+ scriptdev2.conf.dist

Before we start to change configuration settings lets grab our AHBot.conf file 
``bash
cp /root/SOURCES/server/src/game/AuctionHouseBot/ahbot.conf.dist.in /opt/mangos/etc/ahbot.conf.dist
```
I copied ahbot.conf.dist.in to ahbot.conf.dist as i like to have the dist files around in case i need to look at the default settings. We will need to rename them (or simply copy) to reflect name.conf. A little bit of Bash-Fu and we can do all 4 renames in one line:
```bash
# rename the files
ls | sed -e "p;s/\.dist//" | xargs -n2 mv
# to copy and leave the .dist files as a back up as i like to do
ls | sed -e "p;s/\.dist//" | xargs -n2 cp
```
The ls output is piped to sed , then we use the p flag to print the argument without modifications, in other words , the original name of the file.
The next step is use the substitute command to change file extension.
NOTE: We’re using single quotes to enclose literal strings ( the dot is a metacharacter if using double quotes escape it with a backslash).
The result is a combined output that consists of a sequence of old_file_name -> new_file_name.
Finally we pipe the resulting feed through xargs to get the effective rename of the files.

Now that we have adjusted all the file names lets modify a few settings in the files to allow us to connect to the database and get our servers up.
First off lets edit the RealmD settings:
```
# hostname;port;username;password;database
LoginDatabaseInfo = "127.0.0.1;3306;mangos;mangos;realmd"
...
LogsDir = "../logs" # the path is relative to the bin directory
...
# Also because I like colourful readouts to identify problems we specify log colors here
# Format "normal_color details_color debug_color error_color
# The following color codes will be used "13 7 11 9"
LogColors = "13 7 11 9"
...
```

That is the RealmD configuration settings and now for our mangosd configuration:
```
DataDir = "../data"
LogsDir = "../logs"
LoginDatabaseInfo     = "127.0.0.1;3306;mangos;mangos;realmd"
WorldDatabaseInfo     = "127.0.0.1;3306;mangos;mangos;mangos"
CharacterDatabaseInfo = "127.0.0.1;3306;mangos;mangos;characters"
...
# the following are turned on my setup how ever they are not required to turn the server on
WorldLogFile = "world.log"
...
GmLogFile = "gms.log"
...
RaLogFile = "remote_access.log"
...
# Also because I like colourful readouts to identify problems we specify log colors here
# Format "normal_color details_color debug_color error_color
# The following color codes will be used "13 7 11 9"
LogColors = "13 7 11 9"
...
```
and scriptdev2.conf
```
ScriptDev2DatabaseInfo     = "127.0.0.1;3306;mangos;mangos;scriptdev2"

# Log File for SD2-Errors
SD2ErrorLogFile = "../logs/SD2Errors.log" #there is no log directory option so you must specify path + file here
```

and finally the AHBot.conf I will be leaving all values in the AH-Bot at defaults, however i will show you which lines are needed to turn the AH-Bot on:
```
AuctionHouseBot.Seller.Enabled = 1
...
AuctionHouseBot.Buyer.Enabled = 1
```
