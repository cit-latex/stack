# Configure Application Stack (WEB+APP+DB)

## 1. DB Server
```
# yum install mariabd-server -y
# systemctl enable mariadb 
# systemctl start mariadb
```
#### Configure DB 
```
# mysql

> create database studentapp;
> use studentapp;
> CREATE TABLE Students(student_id INT NOT NULL AUTO_INCREMENT,
	student_name VARCHAR(100) NOT NULL,
  student_addr VARCHAR(100) NOT NULL,
	student_age VARCHAR(3) NOT NULL,
	student_qual VARCHAR(20) NOT NULL,
	student_percent VARCHAR(10) NOT NULL,
	student_year_passed VARCHAR(10) NOT NULL,
	PRIMARY KEY (student_id)
);
> grant all privileges on studentapp.* to 'student'@'%' identified by 'student@1';
> flush privileges;
```

## 2. Configure Tomcat 
```
# yum install java -y
# cd /root
# wget -qO- http://www-us.apache.org/dist/tomcat/tomcat-8/v8.5.27/bin/apache-tomcat-8.5.27.tar.gz | tar -xz
# cd apache-tomcat-8.5.27
# rm -rf webapps/*
# wget https://github.com/cit-latex/stack/raw/master/mysql-connector-java-5.1.40.jar -O lib/mysql-connector-java-5.1.40.jar
# wget https://github.com/cit-latex/stack/raw/master/student.war -O webapps/student.war
```
#### Configure context.xml file with DB details
```
# vim conf/context.xml
```
##### Add the following content just before last line
```
<Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource"
               maxTotal="100" maxIdle="30" maxWaitMillis="10000"
               username="student" password="student@1" driverClassName="com.mysql.jdbc.Driver"
               url="jdbc:mysql://<IP-ADDRESS-OF-DB-SERVER>:3306/studentapp"/>
```
#### Start tomcat service
```
# sh bin/startup.sh
```

## 3. Web Server
```
# yum install httpd httpd-devel gcc -y
# cd /root
# wget -qO- http://www-eu.apache.org/dist/tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.42-src.tar.gz | tar -xz
# cd tomcat-connectors-1.2.42-src/native
# ./configure --with-apxs=/usr/bin/apxs
# make 
# make install
# vim /etc/httpd/conf.d/worker.properties
worker.list=tomcatA
### Set properties
worker.tomcatA.type=ajp13
worker.tomcatA.host=<IP-ADDRESS-OF-TOMCAT-SERVER>
worker.tomcatA.port=8009

# vim /etc/httpd/conf.d/mod_jk.conf
LoadModule jk_module modules/mod_jk.so
JkWorkersFile conf.d/worker.properties
JkMount /student tomcatA
JkMount /student/* tomcatA

# systemctl enable httpd
# systemctl start httpd
```

