### Setting the password for the first time   

Open up a terminal window and issue the following command:   
  <code>mysqladmin -u root password NEWPASSWORD</code>

An alternative method for setting the root password for the first time, one that also adds a bit of security to your MySQL database, is to use the mysql_secure_connection command. Not only will this command set the root user password, but it will allow you to remove anonymous users, disallow remote root login, and remove the test database. To use this command, simply type:   
  <code>mysql_secure_connection</code>
  
### Changing the MySQL root user password   
If you've set the password, and want to change it to something different (hint, hint ... more challenging), you can do that. This does require that you know the current password. With that password in hand, the command to change the root user password is:

<code>mysqladmin -u root -p'OLDPASSWORD' password NEWPASSWORD</code>   
In the above command, there is no space between -p and 'OLDPASSWORD'. If you put a space between them, the command will fail.

### Recover MySQL password
1. Stop the MySQL server process with the command <code>sudo service mysql stop</code>
2. Start the MySQL server with the command <code>sudo mysqld_safe —skip-grant-tables —skip-networking &</code>
3. Connect to the MySQL server as the root user with the command <code>mysql -u root</code>

At this point, you need to issue the following MySQL commands to reset the root password:

<pre><code>mysql> use mysql;
mysql> update user set authentication_string=password('NEWPASSWORD') where user='root';
mysql> flush privileges;
mysql> quit</code></pre>
Where NEWPASSWORD is the new password to be used.

Restart the MySQL daemon with the command sudo service mysql restart. You should now be able to log into MySQL with the new password.

