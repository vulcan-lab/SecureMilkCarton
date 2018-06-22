# SecureMilkCarton

SecureMilkCarton is an intentionally vulnerable Tomcat web application. It seems vulnerable web applications for learning _hacking_ or _penetration testing_ are a dime-a-dozen. SecureMilkCarton is different, it has been specifically designed to learn *how to secure a poorly written web application*. The web application itself is riddled with security issues, including:

- Vulberable to SQL injection attacks
- Vulnerable to XSS attacks
- Access control issues
- Bad password storage practices

To add to the problem, the web server used to host the web application also suffers from a collection of security issues, including:

- Non-existant firewall
- Poorly configured MySQL database
- Bad practice implementing security-related HTTP headers
- No HTTPS configured
- Default SSH configuration

SecureMilkCarton was specifically designed as a practical assessment for the Introduction to Information Security course that I teach. This course is situated in the second year of a Bachelor of Information Technology degree, therefore, the assessment targets this specific level. However, the web application could be modified to target a variety of audiences.

Included with SecureMilkCarton are a collection of tasks, somewhat similar to the assessment I wrote for my Introduction to Information Security course. Model answers can be provided to faculty members who are can prove that they are teaching courses in accredited educational institutes. Please email me for additional information.

## SecureMilkCarton: Quick Start

- Install Ubuntu 18.04 server on a VM
- Make sure git is installed:
    - `sudo apt install git`
- Clone this repository to your home directory:
    - `cd ~ && git clone https://github.com/thomaslaurenson/SecureMilkCarton.git`
- Run the web application build script:
    - `cd ~/SecureMilkCarton/build && chmod u+x build.sh && sudo ./build.sh`
    - You will be prompted for the MySQL root user password, use: ``passw0rd`
- Check the web application in a web browser:
    - `<server-ip-address>:8080/securemilk/`
    - For example: `192.168.1.10:8080/securemilk/`

## SecureMilkCarton: Installation

The first step to install SecureMilkCarton is to install and configure a suitable virtual machine. This tutorial follows an virtual machine installed with Ubutnu Linux Server version 18.04 LTS that is hosted on VMWare vSphere (no notable differences in setup compared to VMWare Workstation).

Create a new virtual machine and install using default options.

When installed, make sure to update the opeating system:

```
sudo apt update && sudo apt upgrade
```

Install VMWare tools (if using VMWare):

```
sudo apt install open-vm-tools
```

Clone this repository to your home directory:

```
cd ~
git clone https://github.com/thomaslaurenson/SecureMilkCarton.git
```

From here, you can use the `build.sh` script to install all required packages, configure the services, and then build and deploy the SecureMilkCarton web application.

Make sure you are in the `build` directory of the git repository:

```
cd ~/SecureMilkCarton/build
```

Make sure you have executable rights to the `build.sh` script:

```
chmod u+x build.sh
```

Run the `build.sh` script using sudo rights:

```
sudo ./build.sh
```

### Optional: Install better history

I find when deploying this assessment for learners, it is beneficial to improve the history retention, that is, the information retained by the `~/.bash_history` file and associated `history` command. This project include a simple script (`better_history.sh`) that enhances the history configuration, including:

- Preserves history when multiple different connections to a system are established (e.g., multiple SSH or PuTTY sessions)
- Appends date/time stamps to each history entry

To include better history, execute the following actions to run the script:

```
sudo su
cd ~/SecureMilkCarton/build/
./better_history.sh
exit
```

### Optional: Changing the default ports

It might be easier to change the default Apache Tomcat ports. After a default installation, Apache Tomcat runs on the following ports:

- HTTP on port 8080
- HTTPS on port 8443 (if configured)

For learners new to web applications, and web servers in general, it might be easier to configure Apache Tomcat to operate on default HTTP/HTTPS ports (80/443). The following instructions will make this configuration change using the `authbind` package.

Install the `authbind` package:

```
sudo apt install authbind
```

Switch to the root user:

```
sudo su
```

Make port 80 and port 443 available to authbind, and set the owner to `tomcat` (this account is created during during the building of the web applicaiton):

```
touch /etc/authbind/byport/80
chmod 500 /etc/authbind/byport/80
chown tomcat /etc/authbind/byport/80
touch /etc/authbind/byport/443
chmod 500 /etc/authbind/byport/443
chown tomcat /etc/authbind/byport/443
```

Create a file named `setenv.sh` in the tomcat folder:

```
vim /opt/tomcat/bin/setenv.sh
```

Enter the following content:

```
CATALINA_OPTS="-Djava.net.preferIPv4Stack=true"
```

Open the tomcat start-up script:

```
vim /opt/tomcat/bin/startup.sh
```

Find the following line:

```
exec "$PRGDIR"/"$EXECUTABLE" start "$@
```

And replace the line with:

```
exec authbind --deep "$PRGDIR"/"$EXECUTABLE" start "$@" `
```

Finally, change the tomcat server configuration. Open the `server.xml` file:

```
vim /opt/tomcat/conf/server.xml
```

Find the entry for the service. This should be named `Catalina`, and should look like the output snippet below:

```
  <Service name="Catalina">
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

This configuration file needs to be changed from the default Apache Tomcat ports to the new ports we specified above. The final result will look like the output snippet below. Basically, we just changed port 8080 to 80, and port 8443 to 443.

```
  <Service name="Catalina">
    <Connector port="80" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="443" />
```

Save and exit the files. Then restart the Apache Tomcat service:

```
sudo systemctl restart tomcat
```

Finished.

An alternative method to the using authbind is to utilize `iptables` to forward traffic from the default HTTP/HTTPS ports to the ports tomcat uses. This method is simple, however, I feel it interfers with any `iptables` firewall configuration that can be used.

The following two `iptables` rules will forward all traffic from port 80 and 443 to the default tomcat ports of 8080 and 8443.

```
sudo iptables -t nat -A PREROUTING -i ens160 -p tcp --dport 80 -j REDIRECT --to-port 8080
sudo iptables -t nat -A PREROUTING -i ens160 -p tcp --dport 443 -j REDIRECT --to-port 8443
```

## SecureMilkCarton: Web Server Configuration

An SSH server has been installed using the OpenSSH software. However, no configuration has been performed. You can login to the SSH service on port 22. The login details are any user account that has been created on the virtual machine.

A MySQL database has been installed and partially configured. The credentials for the MySQL database are:

- Username: `root`
- Password: `passw0rd`

Apache TomCat (a type of web server software) has also been installed on the server. The TomCat service is a Java Servlet Container – basically a web server that allows the creation and sharing (via HTTP) of a web application written with Java on the server-side. Although the web application is very simple (compared to real-world web applications), getting used to the setup can take some practice. Here is some relevant information about the web application configuration:

- The web application is deployed in: `/opt/tomcat/webapps/`

## SecureMilkCarton: Project Structure

The directory structure of the web application is very important to learn and understand to make good progress with the assignment. The following directory structure represents the web application:

- `index.jsp`: the HTML code for the login web page
- `search.jsp`: the HTML code for the employee search web page
- `noticeboard.jsp`: the HTML code for the noticeboard web page
- `style.css`: the CSS style sheet for the website
- `jquery.js`: the JQuery library dependency for the noticeboard web page
- `securemilk_logo.jpg`: the company logo to make things look pretty
- `WEB-INF`: a folder containing web application resources
    - `classes`: a folder containing java code
        - `Login.java`: server-side code for the login web page
        - `Noticeboard.java`: server-side code for the noticeboard page
        - `Hashing.java`: server-side code for password hashing
    - `lib`: a folder containing dependencies
        - `mysql-connector-java-5.1.42-bin.jar`: dependency jar file to connect to local MySQL database
        - Other dependencies can be included in this folder
- `database`: a collection of file for database creation and re-creation:
    - `securemilk_db.sql`: a SQL file containing all securemilk database entries
    - `create_db.sh`: this script uses `securemilk_db.sql` to automate creation the database
    - `recreate_db.sh`: this script deletes the old database and recreates the original database using the `securemilk_db.sql` file
    - `passwords.csv`: a comma-separated values (CSV) file containing usernames, passwords and salt values. This file is useful when changing the password hashing algorithm
- `scripts`: a collection of scripts that make life easier:
    - `compile.sh`: a script to compile the Java source code and package the entire web application into a distributable .war file for deployment to the Tomcat server
    - `compile_simple.sh`: the same script contents as `compile.sh` file, but with the code commenting and error checking removed

## SecureMilkCarton: General Usage

This section documents several common scenarios that you will need to perform including creating the database for the web application, deleting the database, compiling the web application and how to access the web application.

### How to create database

The database is essential for the SecureMilkCarton web application to function. Therefore, creating the database is a key step. This process can be automated using the `create_db.sh` script provided in the `SecureMilkCarton/securemilk/databases` folder. Simply run the script using the following command (the following instructions assumes for have the repo in your home folder):

```
cd ~/SecureMilkCarton/securemilk/databases
./create_db.sh 
```

You will be prompted to enter the password for the root user of the database account. If you followed the official documentation, this password will be: `passw0rd`.

The primary action the script performs is depicted in the following code snippet:

```
cat ~/SecureMilkCarton/securemilk/database/securemilk_db.sql | mysql -u root -p
```

This line of code prints the contents of `securemilk_db.sql` file containing the database contents, and pipes the content into the `mysql` command which populates the database.

### How to delete and recreate database

Similar to creating the database, you can also recreate the database using the same `securemilk_db.sql` file. However, be warned... this script drops the entire `securemilk` database and all entries. If you have made any modifications to the database contents, this information will be lost.

Recreating the datadase can be automated using the `recreate_db.sh` script provided in the `SecureMilkCarton/securemilk/databases` folder. Simply run the script using the following command (the following instructions assumes for have the repo in your home folder):

```
cd ~/SecureMilkCarton/securemilk/databases
./create_db.sh 
```

You will be prompted to enter the password for the root user of the database account. If you followed the official documentation, this password will be: `passw0rd`. The first time you are prompted for the password is to drop the database, the second time you are prompted for the password is to create the database again.

### How to compile and deploy

This project uses a simple static technique to compile and deploy a Tomcat web application. This process can be automated using the `compile.sh` script provided in the `SecureMilkCarton/securemilk/scripts` folder. Simply run the script using the following command (the following instructions assumes for have the repo in your home folder):

```
cd ~/SecureMilkCarton/securemilk/scripts
sudo ./compile.sh 
```

Make sure you *use sudo to compile* the web application!

Below is a snippet of the primary code that compiles and deploys the web application, which includes some comments to explain what each line achieves.

```
# Compile each of the three .java files
javac -classpath "/opt/tomcat/lib/servlet-api.jar" ~/securemilk/WEB-INF/classes/Login.java ~/securemilk/WEB-INF/classes/Hashing.java
javac -classpath "/opt/tomcat/lib/servlet-api.jar" ~/securemilk/WEB-INF/classes/Noticeboard.java

# Create a .war file to export to the Java Tomcat web server
jar -cf securemilk.war *

# Copy the .war file to the Java Tomcat web server
sudo cp securemilk.war /opt/tomcat/webapps/
```

### How to access the web application

The web application is accessible through any web browser, however, it is recommended to use Mozilla Firefox or Google Chrome. The specific ports the web application are deployed on are:

- HTTP on port 8080
- HTTPS on port 8443 (not configured by default)

Therefore you can access the web application by using the following URL in your web browser:

- `<server-ip-address>:8080/securemilk/`
- For example: `192.168.1.10:8080/securemilk/`

If you performed the optional step of changing the default ports you can access the web application using the following URL in your web browser:

- `<server-ip-address>/securemilk/`
- For example: `192.168.1.10/securemilk/`

Since SecureMilkCarton is a vulnerable, and terribly configured, web application it is not served using HTTPS by default. If you do implement HTTPS certificates, the web application will then be accessible using the following URLs:

- `https://<server-ip-address>:8443/securemilk/` (if using port 8443)
- For example: `192.168.1.10:8443/securemilk/`
- `https://<server-ip-address>/securemilk/` (if using port 443)
- For example: `https://192.168.1.10/securemilk/`
