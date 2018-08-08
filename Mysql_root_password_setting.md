### Find the initial root password on install
Using following command:   
<code>grep 'temporary password' /var/log/mysqld.log</code>

### Change password policy   
validate_password_policy有以下取值：
* 0 or LOW: Length
* 1 or MEDIUM: Length; numeric, lowercase/uppercase, and special characters
* 2 or STRONG: Length; numeric, lowercase/uppercase, and special characters; dictionary file

默认是1，即MEDIUM，所以刚开始设置的密码必须符合长度，且必须含有数字，小写或大写字母，特殊字符。

有时候，只是为了自己测试，不想密码设置得那么复杂，譬如说，我只想设置root的密码为123456。
必须修改两个全局参数：   
首先，修改validate_password_policy参数的值   
<code>mysql> set global validate_password_policy=0;</code>

这样，判断密码的标准就基于密码的长度了。这个由validate_password_length参数来决定。

validate_password_length参数默认为8，它有最小值的限制，最小值为：
<pre><code>
validate_password_number_count
+ validate_password_special_char_count
+ (2 * validate_password_mixed_case_count)</code></pre>

其中，validate_password_number_count指定了密码中数据的长度，validate_password_special_char_count指定了密码中特殊字符的长度，validate_password_mixed_case_count指定了密码中大小字母的长度。

这些参数，默认值均为1，所以validate_password_length最小值为4，如果你显性指定validate_password_length的值小于4，尽管不会报错，但validate_password_length的值将设为4。如下所示：

<pre><code>
mysql> select @@validate_password_length;
+----------------------------+
| @@validate_password_length |
+----------------------------+
|                          8 |
+----------------------------+
row in set (0.00 sec)

mysql> set global validate_password_length=1;
Query OK, 0 rows affected (0.00 sec)

mysql> select @@validate_password_length;
+----------------------------+
| @@validate_password_length |
+----------------------------+
|                          4 |
+----------------------------+
row in set (0.00 sec)</code></pre>

如果修改了validate_password_number_count，validate_password_special_char_count，validate_password_mixed_case_count中任何一个值，则validate_password_length将进行动态修改。

<pre><code>mysql> select @@validate_password_length;
+----------------------------+
| @@validate_password_length |
+----------------------------+
|                          4 |
+----------------------------+
row in set (0.00 sec)

mysql> select @@validate_password_mixed_case_count;
+--------------------------------------+
| @@validate_password_mixed_case_count |
+--------------------------------------+
|                                    1 |
+--------------------------------------+
row in set (0.00 sec)

mysql> set global validate_password_mixed_case_count=2;
Query OK, 0 rows affected (0.00 sec)

mysql> select @@validate_password_mixed_case_count;
+--------------------------------------+
| @@validate_password_mixed_case_count |
+--------------------------------------+
|                                    2 |
+--------------------------------------+
row in set (0.00 sec)

mysql> select @@validate_password_length;
+----------------------------+
| @@validate_password_length |
+----------------------------+
|                          6 |
+----------------------------+
row in set (0.00 sec)</code></pre>

当然，前提是validate_password插件必须已经安装，MySQL5.7是默认安装的。

那么如何验证validate_password插件是否安装呢？可通过查看以下参数，如果没有安装，则输出将为空。

<pre><code>
mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| validate_password_dictionary_file    |       |
| validate_password_length             | 6     |
| validate_password_mixed_case_count   | 2     |
| validate_password_number_count       | 1     |
| validate_password_policy             | LOW   |
| validate_password_special_char_count | 1     |
+--------------------------------------+-------+
rows in set (0.00 sec)</code><pre>

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

