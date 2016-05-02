### Set Up Tomcat Server
* Remove `apparmor`:
  * `sudo /etc/init.d/apparmor stop`
  * `sudo update-rc.d -f apparmor remove`
  * `sudo apt-get --purge remove -y apparmor apparmor-utils libapparmor-perl libapparmor1`
* Install Tomcat Server (if not installed already):
  * `sudo apt-get install -y tomcat7`
* Stop the Tomcat service:
  * `sudo service tomcat7 stop`
* Remove the `ROOT`  directory:
  * `sudo rm -rf /var/lib/tomcat7/webapps/ROOT`
