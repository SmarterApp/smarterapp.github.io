#### Install MySQL Connector in Tomcat
* Get the mysql-connector-java-5.1.37 tar file:
  * `wget -O /tmp/mysql-connector-java-5.1.37.tar.gz http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.37.tar.gz`
* Change to `/tmp` directory (where the `wget` command above wrote the `.tar.gz` file):
  * `cd /tmp`
* Extract the file(s):
  * `tar xvf mysql-connector-java-5.1.37.tar.gz`
* Change to the mysql-connector directory:
  * `cd mysql-connector-java-5.1.37`
* Copy the `.jar` into Tomcat's `lib` directory:
  * `sudo cp /tmp/mysql-connector-java-5.1.37/mysql-connector-java-5.1.37-bin.jar /usr/share/tomcat7/lib`
* Restart Tomcat:
  * `sudo service tomcat7 restart`