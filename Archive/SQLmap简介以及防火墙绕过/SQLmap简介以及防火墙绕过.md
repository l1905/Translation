![](./img/0.jpg)

##简介
许多现实中对于网站的攻击往往是由于网站没有及时更新或者对于用户的输入没有进行检查。从缓冲区溢出说起，这样一种针对系统脆弱性的威胁，最根本的问题还是在于对于用户的输入没有进行检查。作为主要威胁之一的SQL注入带来了人们对于其应用和数据库的担忧。这个问题的出现有十年的时间了，但是现在仍旧在许多网站中出现。SQL注入就像许多当前主要的的web应用安全攻击问题也被划归在了对于用户输入的检查上。

##正文
许多web开发者并不知道SQL语句能够被自定义（handle）并且假定SQL语句是一个让人信任的命令。这就让SQL语句能够绕过访问控制机制（access control），因此跳过标准的授权和认证检查。甚至有些时候SQL语句能够允许使用服务器操作系统上的命令行的权限。

直接的SQL注入命令是一种方法，它被攻击者构造或者是修改现成的SQL命令来暴露出隐藏的数据或者是覆盖掉有价值的数据，甚至在服务器上执行危险地系统指令

##入门
结构化的语言是数据库的标准声明语言。这让（语言）变的更加简洁并且更易于使用。SQL起源于70年代的IBM实验室。SQL被用来让应用程序与数据库之间进行通信。

SQL使用如下的4种语言来对数据库进行操作

**SELECT** -从数据库中获得一条记录

**INSERT**-向数据库中插入一条记录

**DELETE**-从数据库中删除一条记录

**UPDATE**-从数据库中更新或者改变当前记录

当用户提交的语句被用作数据库的查询语句时SQL就能够发生。比如当用户被网站认证授权，用户发送一个带有"用户名"和"密码"的信息，用户名/密码就与存放在数据库中的内容进行核对。如果相同的话用户就被允许登录，否则用户登录失败。以下是一个在后台进行登录的例子。

####代码

`SELECT * FROM user WHERE username='$username' AND password='$password'`

这个语句简单地指明了从user的表中查询用户名和username相等的，并且密码和password相等的。所以，如果用户发送用户名为"admin",并且密码为"12345",那么SQL语句被创建如下:

`SELECT * FROM user WHERE username='admin' AND password ='12345'`

####注入
那么，如果用户输入' or '1'='1,第一个引号会终止输入的字符串，剩下的会被当做SQL语句，并且在SQL语句中1=1恒为真，这就能够让我们绕过注册机制

`SELECT * FROM user WHERE username='admin' AND password =''or '1'='1' -TRUE`

上述是一个SQL注入的简单地例子，然而在实际过程中，SQL注入比这要复杂的多。在我们的渗透测试中，大多数时候我们有一个非常紧凑的计划表，所以这个时候我们需要一个自动化的攻击来为我们进行注入攻击。

SQLMAP就是一能够利用（系统）脆弱性的工具。它是开源的，并经常被用来在Python写的脆弱的DBMS上进行入侵的渗透测试。它能够检测并利用SQL上的漏洞，让我们举例在操作系统和数据库上使用sqlmap.py。

##一步一步
我将尽可能地以最简洁的方式展示出来。

最常见的检测SQL注入的方法是通过在输入处添加一个单引号(')并且期待（系统）返回一个错误，有些应用程序并不会返回错误。这个时候我们就会利用true/false语句来检查是否这个应用程序会受到SQL注入的攻击。

为了随机的找到还有SQL注入的脆弱性的网站，你能够使用如下格式的语句利用google dork：**inurl:news.php id =1?**将会出现一堆google dork的数据并且为你过滤你搜索得到的结果提供了可能。那么让那个我们开始吧。

首先在backtrack上进入目录：

`cd /pentest/database/sqlmap`

我们不会立即开始，查看sqlmap.py的菜单使用命令：

./sqlmap.py -h

让我们运行sqlmap.py, 参数是[ --dbs],来发现DBMS中的所有数据库

![](./img/1.jpg)

或者是使用参数 --current-db来发现当前目标使用的数据库

![](./img/2.jpg)

参数-D表示目标的数据库，--tables显示表列

![](./img/3.jpg)

![](./img/4.jpg)

我们将会检查在信息中是否包含感兴趣的内容(admin_users),并把它们按列显示出来，参数是--columns

![](./img/5.jpg)

在你列出表格之前每次指定目标数据库(使用-D参数)是很重要的，因为如果没有-D参数，程序将列出数据库中所有的表格

![](./img/6.jpg)

-T =目标参数

-C=目标列（可以指定超过一列，如：用户名(列),密码(列))

--dump=获得，提取数据

![](./img/7.jpg)

使用--proxy参数来使用代理

例如:./sqlmap.py --url "http://testphp.vulnweb.com/listproducts.php?cat=1" --dbs --proxy=http://183.223.10.108:80

![](./img/8.jpg)


我认为以上是对于初学者的基础的命令。sqlmap同样提供许多有趣的功能，我建议使用--prefix=PREFIX，--postfix=POSTFIX和takeover选项。更多关于该工具的使用可以访问官方网站。

![](./img/9.jpg)

--dump是用来提取网站上的数据，调用时必须选中列，并且你必须明确从列中具体提取什么内容，这里我提取列中保存的登录和密码信息。

通常，DBMS的"password"字段是加密的。通常使用的加密算法是SHA-1，MD5,这些算法在使用时并没有加入"salt"(指根据用户的输入直接进行算法计算),这就使得破解更加容易。那么(拿到加密的数据后)我们就需要对其进行解密，我们能够使用许多在线解密网站如:

http://www.md5decrypt.org,

https://crackstation.net/,

http://www.onlinehashcrack.com/

或者尝试手工暴力破解和彩虹表。此外你还能够使用你的GPU来加快(破解)的过程，这就不是本文所要主要讨论的内容了。

##进阶
幸运的是，sqlmap有一些非常好的脚本，在如下的地址中你能够发现它们。使用svn检查

https://svn.sqlmap.org/sqlmap/trunk/sqlmap sqlmap-dev

事实上，脚本的作用是修改我们发出的请求来防止其被WAF(网络应用防火墙)拦截。在某些情况你可能需要把一些脚本合并到一起才能过WAF。脚本的完整列表访问如下:

https://svn.sqlmap.org/sqlmap/trunk/sqlmap/tamper/

许多企业经常忽视当前(DBMS)的脆弱性而依赖于网络防火墙。不幸的是，经过简单地代码编码就能绕过大部分防火墙。所以先生们，我想展示一下如何利用一些新功能来绕过WAF/IDF(入侵检测系统）。

我将展示一些重要的脚本如charencode.py和charcodeencode.py来与MySQL进行操作，这些脚本能够在backtrack5的
/pentest/web/scanners/sqlmap下面找到。

![](./img/10.jpg)

Hands-on:在你使用这些脚本的时候，使用--tamper参数后面跟脚本名字，在截图中我们使用了charencode命令

![](./img/11.jpg)

**charencode.py总结**

简单的说，这个脚本能够绕过一些比较简单的网络防火墙(WAF).. 其它的比较有趣的功能是(WAF)在匹配它们的规则之前会对url进行解码。web服务器会无如何到url解码的位置，这就应该能够试用与任何一个DBMS(原文是:The web server will anyway go to url-decoded back version,concluding,it should work against and DBMS)

![](./img/12.jpg)

例:

![](./img/13.jpg)

另一个好的脚本是charunicodeencode.py,在我的实际渗透测试过程中，它帮助我绕过了许多防火墙的限制。

![](./img/14.jpg)

嘿，我只是展示了一小部分脚本，我强烈建议你将每个都使用一遍，因为它们往往适用于不同的环境。

**注意：这并不是脚本小子的做法，负责地，熟练地掌握这样一个强大的工具是非常重要的**

##匿名
我将想你展示怎么样使用sqlmap和Onion路由器来保护你的IP，DNS等等，在linux中，在终端命令符为$时使用
sudo apt-get install tor tor-geoip

进入sqlmap的目录后:`./sqlmap.py -u "http://www.targetvuln.com/index/php?cata_id=1" -b -a -tor --check-tor--user-agent="Mozilla/5.0(compatible;Googlebot/2.1;+http://www.google.com/bot.html)"`


参数--tor使用Tor，--check-tor会检查Tor是否被正确地使用，如果没有正确被使用，终端会提示错误信息。用户代理是googlebot，所有你的请求会被看起来像是Googlebot发出的一样。

利用sqlmap的Tor我们能够设置你的TOR代理来隐藏实际请求产生的地址

-tor-port，-tor-type：这两个参数能够帮你手动设置TOR代理，--check-tor参数会检查你的代理是否被正确地安装并正常的工作。

##结语
当SQL注入在几年前被发现时，许多目标都存在这样的缺陷,注入的格式就是其中最难的部分。渗透测试者往往需要自己构造这样的SQL语句。

接下来的发展就产生了自动注入的工具。当前最知名的工具可能就是sqlmap.py。SQLMAP是用python写的开源的测试框架，支持MySQL，Oracle，PostgreSQL，Microsoft SQL Server，Microsoft Access，IBM DB2，SQLite，Firebird，Sybase，SAP，MAXDB并支持6中SQL注入手段。

##解决方案

1.定期检查SQL服务器（的处理请求）

2.限制动态SQL语句

3.避免从用户直接获得数据

4.将数据库的权限信息单独存放在另外一个文件中

5.使用最小权限原则

6.使用预先准备好的(SQL)语句

在此非常感谢我的哥哥Rafay Baloch和RHA的同仁们。

*来源:
http://www.rafayhackingarticles.net/2014/03/introduction-to-sqlmap-and-firewall.html*

#原文


![](./img/0.jpg)

##ABSTRACT

Most cyber-attacks in the world that involve websites occurs due to lack of updates and the failure to validate the user input. Starting from buffer overflow vulnerability, which is a system level vulnerability up to the vulnerabilities that exist today, the fundamental problem has always been the input validation. One of the main threats is SQL Injection that left many worried about their application and databases. The problem is more then a decade old, but still is present inside lots of websites. SQL injection like all other major web application security problems fall in the category of input validation attacks.

##CONTEXT

Many web developers do not know how SQL queries can be handled and assume that an SQL query is a trusted command. This allows for SQL queries to circumvent access controls, thereby bypassing standard authentication and authorization checks. And sometimes SQL queries even may allow access to the command shell on the server operating system level.

Direct injection of SQL commands is a technique where an attacker creates or alters existing SQL commands to expose hidden data or to override valuable data, and even to execute dangerous system level commands on the server.

##INTRODUCTION

Structured Query Language is the standard declarative language for relational databases. This allows for its simplicity and ease of use. SQL was originally developed in the early 70s at IBM labs. SQL Is used by applications communicate or speak with the database.

SQL use the following four statements to communicate with the database.

**SELECT** – Retrieve a record from the database.

**INSERT** -  Inserting a record inside the database.

**DELETE** – Deleting a record from database.

**UPDATE**  - Update or change a current record

The  SQL Injection simply occurs when the user query is treated as the database query. Take an example of how a user is authenticated on a website. The user sends a "Username" and "Password", the username/password is checked against what is present inside the database. If that matches, the user is logged in, else the user is not logged. Here is an example of the basic SQL query that is constructed at the back end.

**The Code:**

`SELECT * FROM user WHERE username = ‘$username’ AND password = ‘$password' `

The query simply says select all from the user table where the username is equal to what the user entered and the password being that the user entered. So, in case where the user supplies the username to be "admin" and password to be "12345". Here is how the SQL query is contructed.

`SELECT * FROM user WHERE username = ‘admin’ AND password = ‘12345' `

**The Injection**

So, what if a user enters something like ' or '1'='1, since the first quote would terminate the input string, the rest of the query would be executed as a SQL query and in a SQL query 1=1 is always a true statement. Which would enable us to bypass the login mechanism

`SELECT * FROM user WHERE username = ‘admin’ AND password = ‘’ or ‘1’=‘1' – TRUE `

The above demonstrates a simple example of SQL injection, however SQL injection in real world is really complicated sometimes and in our penetration tests, most of the times we have a very tight schedule. So therefore, we need an automated/context aware tool to perform the injection for us.

SQLMAP is a tool that can be used to exploit this type of vulnerability. It is Open source, and often is used for Penetration Testing that enable intrusions on fragile DBMS written in Python. It provides functions to detect and exploit vulnerabilities of SQLI. Let's use the example sqlmap.py, widely used in operating systems and databases.

##STEP BY STEP

Readers I will try to explain this in the simplest possible way.

The most common way of testing a SQL injection vulnerability is by inputting a single quote (') inside the input and expect it to display an error. Some of the applications do not return an error. This is where we can utilize true/false statements to check if an application is vulnerable to a SQL injection attack or not.

To find a random website vulnerable to SQL injection, you can utilize the following google dork Example: inurl: news.php id = 1?
There is a bank of google dorks data and several other possibilities that can be used to filter your search. So let's start with SQL map.

Navigate to the following directory inside of backtrack:

`cd /pentest/database/sqlmap`

We will now begin the game, to view the menu for sqlmap.py use the command ./sqlmap.py -h

Let's run sqlmap.py, the parameter [--dbs], to search the all databases in DBMS.


![](./img/1.jpg)

Or use the parameter --current-db to show the databases that are being used. 

![](./img/2.jpg)

The parameter -D is for the target of database and --tables is tables list. 

![](./img/3.jpg)

![](./img/4.jpg)

We will verify the existence of interesting information in the table (admin_users), time to list the columns. The parameter is –columns. 

![](./img/5.jpg)

It is important to always indicate the target database (-D) data before listing the tables because if you do not do this (without the -D) it will list all tables in all databases. 

![](./img/6.jpg)

-T = target table 

-C = target columns, can be more than one column to be chosen. Example: username, password. 

--dump = obtain, extract data. 

![](./img/7.jpg)

Important to remember the parameter --proxy: enables use of proxy. 

Example:     /sqlmap.py --url "http://testphp.vulnweb.com/listproducts.php?cat=1" --dbs --proxy=http://183.223.10.108:80 

![](./img/8.jpg)

Readers, I think that's the basics for beginners. sqlmap.py also has many interesting functions, I suggest researching about --prefix=PREFIX, --postfix=POSTFIX and takeover options. More information about the program and videos of them in action on the official site. 

![](./img/9.jpg)

--dump is to extract the data from the site but is not given any, this must be within the selected column, and you have to choosen what to extract from the column, where I extracted the logins and passwords are saved within the column.

Generally, the field of "passwords" DBMS are encrypted. Most common encryptions used are SHA-1, MD5 hashes and most of the time the hashes are not salted, making it easier to crack. We then need to decrypt the passwords in order to access the target system. We can utilize online hash databases such as  http://www.md5decrypt.org, https://crackstation.net/,http://www.onlinehashcrack.com/ to name a few. The second option is to manually try bruteforcing the password or utilize a commonly known technique rainbow tables for cracking the password hashes. Furthur more you can also try utilizing your GPU power to fasten the process, but unfortunately that's not the scope of this article. 

##BEYOND THE BASICS


Readers, lucky for us, there are some awesome tamper scripts for sqlmap, which can be found in the latest development version from the Subversion repository.

svn checkout https://svn.sqlmap.org/sqlmap/trunk/sqlmap sqlmap-dev

In fact the function of the tamper scripts is to modify the request in a way that will escape detection rules WAF (Web Application Firewall). In some cases it may be necessary to combine some tamper scripts together in order to fool the WAF. For a complete list of scripts for tampering, you may find https://svn.sqlmap.org/sqlmap/trunk/sqlmap/tamper/

Many enterprises often overlook the current vulnerabilities and rely only on the firewall for protection. Unfortunately, most, if not all firewalls can be bypassed by simply utilizing some of the encoding techniques that a sql server understands. So gentlemen, I want to demonstrate how to use some of the new features of sqlmap to bypass WAF’s/IDS.

Well, I'll demonstrate some important scripts that are charencode.py and charcodeencode.py to work with MySQL. The scripts can be found inside of /pentest/web/scanners/sqlmap directory inside of backtrack 5. 

![](./img/10.jpg)

Hands-on: To begin using tamper scripts, you use the --tamper followed by the script name. Inside the screenshot, we have used the charencode command. 

![](./img/11.jpg)

**Summary of charencode.py**

Quite simply, this script is useful for ignoring very weak web application firewalls (WAF) … Another interesting function url-decode the request before processing it through their set of rules (: The web server will anyway go to url-decoded back version, concluding, it should work against any DBMS. 

![](./img/12.jpg)

Example to use:

![](./img/13.jpg)

Another great script is the **charunicodeencode.py**, during my penetration tests this particular script has really helped me in bypassing lots of firewall restrictions. 

![](./img/14.jpg)

Guys, I have demonstrated just a few of the many tamper scripts. We highly recommend testing them out as each one can be used in different situations.

**Notes: That's not a tool for "script kiddies" it is of utmost importance to make use of such a powerful tool responsibly and maturely.**

Caution if used in the wrong way, sqlmap generates many queries and can affect the performance of the database target, moreover strange entries and changes to the database schema are possible if the tool is not controlled and used extensively.

##PARTLY ANONYMOUS


Gentlemen I will demonstrate to you how to use sqlmap with The Onion Router for the protection of IP, DNS, etc... In your Linux, in the terminal type:
$ sudo apt-get install tor tor-geoip

After enter the sqlmap folder and type:
`./sqlmap.py -u "http://www.targetvuln.com/index.php?cata_id=1" -b -a –tor --check-tor--user-agent="Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)"`

The argument --tor invokes the Tor to be used and the --check-tor checks if Tor is being used properly, if not, you will receive an error message in red at the terminal. The User Agent is the googlebot, all your requests on the site will look like the Google bot doing a little visit.

TOR at SQLMap, we can set your TOR proxy for hiding the source from where the traffic or request is generated.

**–tor-port, –tor-type** :  the parameter can help you out to set the TOR proxy manually.
–check-tor : the parameter will check if the tor setup is appropriate and functional.

##CONCLUSION


It is known that many targets have been explored through SQL Injection a few years ago when this threat was discovered, the injection form was "the nail". The pentester had to construct the SQL queries manually.

Then came the development of programs that automated attack. Nowadays perhaps the best known of these programs is sqlmap.py. SQLMAP is a program of open source testing framework written in Python. It has full support for database systems: MySQL, Oracle, PostgreSQL, Microsoft SQL Server, Microsoft Access, IBM DB2, SQLite, Firebird, Sybase, SAP MaxDB and also supports 6 types of SQL Injection techniques.

##SOLUTION


1. Correct the SQL server regularly.
2. Limit the use of dynamic queries.
3. Escape input data from users.
4. Stores the credentials of the database in a separate file.
5. Use the principle of least privilege.
6. Use prepared statements

A Big hug for my dear brother Rafay Baloch and all followers of the RHA. 




