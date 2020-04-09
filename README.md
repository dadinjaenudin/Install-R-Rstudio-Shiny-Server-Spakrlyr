# Install-R-Rstudio-Shiny-Server-Spakrlyr
How to install R, Rstudio, Shiny Server and Sparklyr

## Install R
```
Install R first

Update: Finally I've resolved myself the problem updating the RHEL repo: 
cd /etc/yum.repos.d/
vi CentOS-base.repo
[base]
name=CentOS-$releasever – Base
baseurl=http://buildlogs.centos.org/centos/7/os/x86_64-20140704-1/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
priority=1
exclude=php mysql

And after that: 
$ yum update
$ cd /root/source/R
$ yum install R \
texinfo-tex-5.1-4.el7.x86_64.rpm  \
texlive-epsf-svn21461.2.7.4-43.el7.noarch.rpm \
texinfo-5.1-4.el7.x86_64.rpm \
tre-devel-0.8.0-10.el7.art.x86_64.rpm  \
blas-devel-3.4.2-4.el7.x86_64.rpm \
blas-3.4.2-4.el7.x86_64.rpm \
lapack-devel-3.4.2-4.el7.x86_64.rpm  \
lapack-3.4.2-4.el7.x86_64.rpm \
lapack-devel-3.4.2-4.el7.x86_64.rpm  \
tre-0.8.0-10.el7.art.x86_64.rpm

yum remove R-devel.x86_64 \
R.x86_64 \
R-java-devel.x86_64	3.2.3-4.el7 \
R-java.x86_64 \
R-core.x86_64 \
R-core-devel.x86_64	\
libRmath-devel.x86_64 \
libRmath.x86_64


You'll also need to install the Shiny R package and some other dependencies before installing Shiny Server:

$ yum install libcurl-devel.x86_64
$ su - -c "R -e \"install.packages(c('shiny', 'rmarkdown', 'devtools', 'RJDBC'), repos='http://cran.rstudio.com/')\""

$ R
q()

Uninstall R
Suppose you install R using yum, then you can use the following commands to totally uninstall R:

yum remove R
yum remove R-core
yum remove R-devel
yum remove R-core-devel
```

## Install Shiny Server
```
All the dependencies required for the Shiny server are installed now and we are ready to install it on CentOS 7. So let's download the latest stable release of the Shiny server using the following command.

$ wget https://download3.rstudio.org/centos6.3/x86_64/shiny-server-1.5.9.923-x86_64.rpm

Next, install the downloaded shiny server using the following command.
$ yum install --nogpgcheck shiny-server-1.5.9.923-x86_64.rpm

Note Error :
/var/tmp/rpm-tmp.9F5O5V: line 14: initctl: command not found
/var/tmp/rpm-tmp.9F5O5V: line 17: initctl: command not found

Solution
$ sudo cp /opt/shiny-server/config/systemd/shiny-server.service /etc/systemd/system/
$ sudo systemctl enable shiny-server
$ sudo systemctl restart shiny-server

Web Interface

Open up your favorite web browser and visit http://172.16.7.115:3838/ 
then you'll see a welcome webpage of Shiny server like this:

If you see the above-given output then the Shiny server is successfully installed on your CentOS 7 system. Now you'll need to start and enable the shiny server services. Execute the following commands and they'll do the job for you.

$ systemctl start shiny-server
$ systemctl enable shiny-server

Finally, you'll need to modify the firewall rules for your Shiny server to access it. Shiny server listen to port number 3838 by default.
firewall-cmd --permanent --zone=public --add-port=3838/tcp
firewall-cmd --reload

```

## Install RStudio
```
Installing R Studio Server
Installing R Studio on the same local network as the Spark cluster that we want to connect to - in our case directly on the master node - is the recommended approach for using R Studio with a remote Spark Cluster. Using a local version of R Studio to connect to a remote Spark cluster is prone to the same networking issues as trying to use the Spark shell remotely in client-mode (see part 2).

First of all we need the URL for the latest version of R Studio Server. Preview versions can be found here while stable releases can be found here. At the time of writing Sparklyr integration is a preview feature so I’m using the latest preview version of R Studio Server for 64bit RedHat/CentOS (should this fail at any point, then revert back to the latest stable release as all of the scripts used in this post will still run). Picking-up where we left-off in the master node’s terminal window, execute the following commands,

Can look for the latest available version here.
$ wget https://download2.rstudio.org/rstudio-server-rhel-1.1.463-x86_64.rpm
$ yum install rstudio-server-rhel-1.1.463-x86_64.rpm -y
$ yum install openssl-devel
$ yum install libxml2-devel.x86_64
$ yum install libcurl-devel.x86_64

Add some R libraries (could be done after Rstudio install)
$ R -e "install.packages('devtools', repos = 'http://cran.us.r-project.org')"
$ R -e "install.packages('ggplot2', repos = 'http://cran.us.r-project.org')"


Creating RStudio User
It is not advisable to use the root account with RStudio, instead, create a normal user account just for RStudio. The account can be named anything, and the account password will be the one to use in the web interface.
(requires a UID > 500)

$ adduser rstudio
$ passwd rstudio

Open Browser 
http://172.16.7.115:8787
Login : rstudio
passw : rstudio

Changing the default port for Rstudio Server
• By default it is set to 8787, but this is not exported from
• So, create/edit /etc/rstudio/rserver.conf and add the

single line www-port = 8090

Startup Rstudion
$ ps -ef | grep rstudio
root      9331     1  0 Jan07 ?        00:00:00 /usr/lib/rstudio-server/bin/rserver
$ kill -9 9331
$ rstudio-server start

Automating the starting of RStudio Server
• Create a script Rstudio in /etc/init.d
#!/bin/sh
/usr/lib/rstudio-server/extras/init.d/redhat/rstudio-server start
Don’t forget to run
chmod +x /etc/init.d/RStudio

Stop 
$rstudio-server stop

/*=================================*/
/* Error R initializing */
/*=================================*/

cd /home/akrian
sudo chown -R akrian:akrian .rstudio

1) mkdir /home/biology/.rstudio  
2) mkdir /home/biology/.rstudio/graphics-r3  
3) sudo chown -R biology:my_group .rstudio  

```

## Install Sparklyr
```
Open RStudio http://172.16.7.115:8787/

# Sys.setenv(http_proxy="172.16.13.62:8080")
# Sys.setenv(https_proxy="172.16.13.62:8080")

# Please make sure yum install openssl-devel libxml2-devel.x86_64 libcurl-devel.x86_64
install.packages(c('devtools'), repos='http://cran.rstudio.com/')
devtools::install_github("hadley/devtools")
devtools::install_github("r-lib/xml2")
devtools::install_github("rstudio/sparklyr")
devtools::install_github("tidyverse/dplyr")

# Start RSutdio
$ rstudio-server restart
$ ps aux | grep myuser
```

