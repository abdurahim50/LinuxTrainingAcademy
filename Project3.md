## Installing a LAMP Web App: HESK

Goal:
The goal of this project is to deploy the HESK
application. It's help desk software that is written in
PHP, so you'll configure the LAMP stack for this web application.

Instructions:
**Create a Virtual Machine**
First, start a command line session on your local machine. Next, move into the working folder you created for this course.
```
cd linuxclass
```
Initialize the vagrant project using the usual process of creating a directory, changing into that directory, and running "vagrant init". We'll name this vagrant project "hesk".

```
mkdir hesk
cd hesk
vagrant init jasonc/centos8
```
**Configure the Virtual Machine**
Edit the Vagrantfile and set the hostname of the virtual machine to "hesk". Also, assign the IP
address of 10.23.45.20 to the machine.

```
config.vm.hostname = "hesk"
config.vm.network "private_network", ip: "10.23.45.20"
```
**Start the Virtual Machine**
Now you're ready to start the VM and connect to it.

```
vagrant up
vagrant ssh
```

**Install Apache**

Start off by installing the Apache HTTP Server.

```
sudo dnf install -y httpd
Install PHP
```

This web application is written in PHP, so you need to install PHP. Also, the application will be storing information in a database. This means we'll also need the php-mysqlnd package.

```
sudo dnf install -y php php-mysqlnd
Start and Enable the Web Server
```

Now that the web server and PHP are installed, we can go ahead and start it. We want it to start on boot, so we'll enable it as well.

```
sudo systemctl start httpd
sudo systemctl enable httpd
```

**Install MariaDB**

Next, install MariaDB. While you're at it start and enable it, too.

```
sudo dnf install -y mariadb-server
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

**Secure MariaDB**

Let's secure the default MariaDB installation. The only information you need to supply is the password for the root database user. For our purposes of practicing our skills we can use a simple password such as "root123". In the real world I would suggest using a stronger password. For the remaining questions you can accept the defaults by pressing ENTER.

```
sudo mysql_secure_installation
```
Example:

```
...
Enter current password for root (enter for none): (press ENTER)
Set root password? [Y/n] (press ENTER)
New password: root123
Re-enter new password: root123
Remove anonymous users? [Y/n] (press ENTER)
Disallow root login remotely? [Y/n] (press ENTER)
Remove test database and access to it? [Y/n] (press ENTER)
Reload privilege tables now? [Y/n] (press ENTER)

```
**Create a Database for the Application**

Use the mysqladmin command to create a database named "hesk". Enter the root password when prompted.

```
mysqladmin -u root -p create hesk
```
**Create a DB User for the Database**

First, connect to the MariaDB server using the mysql client. Next, use the GRANT command to create the user, set the password, and allow full permissions to the "hesk" database. Name the user "hesk" and set the password to "hesk123". Remember to flush the privileges.

```
mysql -u root -p
> GRANT ALL on hesk.* to hesk@localhost identified by 'hesk123';
> FLUSH PRIVILEGES;
> exit

```


**Download the Web Application**

Now, download the web application. You can do that directly from your Linux system by using the curl command. Curl is mostly used to transfer data over a network such as downloading a file.The "-L" option tells curl to obey redirects. Use the "-O" option causes the file to be saved locally with the same name that was used on the remote system.

By the way, when you specify multiple options to a command you can specify them separately like this: curl -L -O. However, when those options aren't expecting anything following them you can use a single hyphen and then combine all the single letter options like this:curl -LO. The order doesn't matter either, so you can do this as well: curl -OL. So, curl -L -O, curl -O -L, curl-LO, and curl -OL are all the same.

Remember to type in the entire command. Copying and pasting from PDFs, web pages, and other documents may cause problems. For example, you may see a hyphen, but actually what is in the document is "pretty" character that represents that hyphen. If you were to copy and paste the contents in a terminal, it will not understand that character.

```
curl -LO http://mirror.linuxtrainingacademy.com/hesk/hesk.zip
```

Internet download location:
https://www.hesk.com/download.php


**Extract the Web Application into the DocumentRoot**

Now that you've downloaded the web app, it's time to extract its contents into the DocumentRoot.By default, the DocumentRoot is /var/www/html, so that's where you'll place the files. Just like installing software with packages, putting files in most locations requires root privileges. Use sudo in conjunction with the unzip command to perform this task. Confirm the move with ls.

```
cd /var/www/html
sudo unzip /home/vagrant/hesk.zip
```

**Assign File Permissions for the Web Application**

In a later step, you will use the web application's web installer. You will answer some questions and the web application will write a configuration file based on those answers. In order to allow it to write to the file, it needs permissions. The web server runs as the "apache" user, so one simple way to give the web application permission is to change the ownership of the configuration file to "apache". Do that with this command:

```
sudo chown apache hesk_settings.inc.php
```

(NOTE: The full path to the file is /var/www/html/hesk_settings.inc.php).

Next, we need to give the web application permission to save attachments. For example, someone who submits a ticket might include a screenshot, and we need a place to store that attachment. Run this command to change the ownership of the "attachments" directory to "apache".

```
sudo chown apache attachments
```

(NOTE: The full path to the directory is /var/www/html/attachments).

Next, we need to give the web application permission to save cached files. Each time a PHP page is accessed, PHP code will execute to generate HTML that will be delivered to the site visitor. In certain cases, the execution of the PHP code can take a relatively long time. This application, like many others, can save the resulting HTML to disk and serve that static file to visitors instead of executing the PHP code every time the page is visited. This technique is called caching. So, lets allow the web application to write files into the cache directory.

```
sudo chown apache cache
```

(NOTE: The full path to the directory is /var/www/html/cache).

This application comes with a set of email templates that can be customized from the web interface. To allow the web application to save those customizations, change the owner of the "language/en/emails" directory to "apache".

```
sudo chown apache language/en/emails
```

(NOTE: The full path to the directory is /var/www/html/language/en/emails).

**Complete the Web Application Install**

Open up a web browser on your local machine and visit http://10.23.45.20/install. Be sure to append "/install" after the IP address of the VM.

Click the button labeled "Click here to INSTALL HESK".

Click the button labeled "I ACCEPT (Click to continue)".

(NOTE: If you did not correctly configure the permissions you will get an error message. In order to continue with the installation, you will need to perform the steps from the "Assign File Permissions for the Web Application" section above.)

Now you should be on the "Database settings" page.

Use the default value for "Database Host", which is "localhost".

Use the default value for "Database Name", which is "hesk".

Change the "Database User (login)" field from "test" to "hesk".

Change the "User password" field from "test" to "hesk123".
Use the default value for "Table prefix", which is "hesk_".

Change the "Choose a Username" field from "Administrator" to "admin".

Change the "Choose a Password" field from blank to "admin".

OPTIONAL: Change the "Help desk timezone" field from "UTC" to your time zone. (America/New_York, for example.)

Click the button labeled "Continue to Step 4".

**Potential Troubleshooting Required**

At this point the web application will attempt to connect to the database. If it can't connect to the database, you will get an error message that states something like "Database connection failed:Double-check all the information below." This means there is a mismatch between the database username and password the application is using and the username and password set on the database.

In your web browser, make sure that the "Database User" is set to "hesk" and the "User Password" is set to "hesk123". Click the button labeled "Continue to Step 4" again.

If you still get an error, return to the command line and re-run the database GRANT statement. Remember, the password for the root database user is "root123".

```
mysql -u root -p
> GRANT ALL on hesk.* to hesk@localhost identified by 'hesk123';
> FLUSH PRIVILEGES;
> exit
```
Return to your web browser and click the button labeled "Continue to Step 4" again.

**Complete the Web App Install**

No you will be on the "Finishing touches" page. The web application no longer needs its install directory, so we'll remove it. Return to the command line and type:

```
sudo rm -rf /var/www/html/install
```
Return to the web browser and click the button labeled "Skip directly to settings."

**Configure the Web Application**

You'll now be looking at the General settings page. To change these settings, you have to edit the /var/www/html/hesk_settings.inc.php file. Just to prove this is where the settings are stored, we're going to change the "Website title."

Return to the command line and make a backup copy of the file you are about to edit.

```
sudo cp hesk_settings.inc.php hesk_settings.inc.php.bak
```

(NOTE: You should still be in the /var/www/html directory. If you are not, change there before running the above command. IE: cd /var/www/html)

Use your favorite text editor to open the file.

```
sudo nano hesk_settings.inc.php
```

Find this line in the file:

```
$hesk_settings['site_title']='Website';
```

Replace "Website" with "Initech". The line should look like this:

```
$hesk_settings['site_title']='Initech';
```

Return to your web browser and reload the page. (Ctrl-R on Windows, Cmd-R on Mac.) You should now see the change.

**WARNING:** Be careful with your typing! If you accidentally delete a semicolon or make some other typing mistake, it could cause the web app to become unusable.If you run into an error and can't find your mistake, restore the backup of the configuration you made earlier by running:

```
sudo cp hesk_settings.inc.php.bak hesk_settings.inc.php
```

**The Web App is Ready**

Now you can start using the application. Feel free to explore the Administration Control Panel at http://10.23.45.20/admin/ or find out what a normal user would experience by visiting http://10.23.45.20/.

Even though you can start using this web application, the goal was to learn how to deploy the application on a Linux system. I'll leave it up to you as to how much you would like to explore the application.

In the next lesson you will learn how to configure the system to send email notifications.


http://www.LinuxTrainingAcademy.com

