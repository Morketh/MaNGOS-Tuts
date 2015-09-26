#INSTALL NOTES FOR CLEAROS MaNGOS Core 3

```bash
yum-config-manager --enable clearos-developer clearos-epel clearos-core 
yum upgrade && yum install clearos-devel app-devel
yum --enablerepo=clearos-updates-testing upgrade
yum install mysql-devel
yum install bzip2-devel
yum install pv
mkdir SOURCES && cd SOURCES
git clone https://github.com/CollectiveIndustries/server.git
git clone https://github.com/CollectiveIndustries/database.git
git clone https://github.com/CollectiveIndustries/scripts.git server/src/bindings/scripts
git clone https://github.com/CollectiveIndustries/mangos-enhanced.git /var/www/html
git clone https://github.com/CollectiveIndustries/mangos-enhanced.git /var/www/html/mangos

cmake .. -DTBB_USE_EXTERNAL=1 -DCMAKE_BUILD_TYPE=release -DACE_USE_EXTERNAL=0 -DCMAKE_INSTALL_PREFIX=/opt/mangos -DINCLUDE_BINDINGS_DIR=scripts -DPCH=0 -DCMAKE_CXX_FLAGS="-O3 -march=native" -DCMAKE_C_FLAGS="-O3 -march=native"
```