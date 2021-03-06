---
layout: post
title: SQL Injections
slug: sql-injections-old
author: Martin Thoma
date: 2013-07-11 19:25:03.000000000 +02:00
category: Code
tags: PHP, Database, IT-Security, MySQL, SQL
featured_image: 2013/07/blackhat.png
---
<abbr title="Structured Query Language">SQL</abbr> is a language that allows prorammers to access data in databases. Most of the time (always?) you pass your queries in form of strings to the database. In online services it is quite common that the programmer formulates a template and the user fills in variables.

<h2>Example: IMDb</h2>
Take a look at <a href="http://www.imdb.com/">IMDb</a>. Users can search for movies by title:

<figure class="aligncenter">
            <a href="../images/2013/07/imdb-harry-potter-query.png"><img src="../images/2013/07/imdb-harry-potter-query.png" alt="IMDb: User Interface to search for a movie by title" style="max-width:300px;max-height:194px" class="size-medium"/></a>
            <figcaption class="text-center">IMDb: User Interface to search for a movie by title</figcaption>
        </figure>

When you search for "Harry Potter" for example, the following happens:

<figure class="aligncenter">
            <a href="../images/2013/07/imdb-search-query-300x156.png"><img src="../images/2013/07/imdb-search-query-300x156.png" alt="Search query within URL and results" style="max-width:300px;max-height:156px" class="size-medium wp-image-73921"/></a>
            <figcaption class="text-center">Search query within URL and results</figcaption>
        </figure>

You obviously interacted with imdb.com in a very dynamic way. The output of the website depends on what you typed in and IMDb has to search in its database for your search terms.

The programmers might have created a query that looks like this

```sql
SELECT * FROM `movie` WHERE title=$_GET['q']
```

Where <code>$_GET['q']</code> is your query.

<h2>Proof of concept</h2>
You need:
<ul>
  <li>Apache Web Server</li>
  <li>PHP</li>
  <li>MySQL</li>
  <li>PhpMyAdmin (for convenience)</li>
</ul>

When you search for "LAMP" (for Linux users) or for "WAMP" (for Windows users) you find a lot of information how to install this.

Place the following as <code>hack.php</code> in your web servers directory (might be <code>/var/www</code>):

```php
<?
$mysqlhost = "localhost";
$mysqluser = "root";
$mysqlpwd = "asdfasdf";
$connection = mysql_connect($mysqlhost, $mysqluser, $mysqlpwd) or die
("Your connection string was wrong");
$db_selected = mysql_select_db('imdb', $connection);

$q = $_GET['q'];
if ($q != "") {
    $result = mysql_query("SELECT * FROM `movies` WHERE title='$q'");
    if (!$result) {
        die('MySQL query error: ' . mysql_error());
    }
    echo "Found ".mysql_num_rows($result)." movies:<br/>";
    while ($row = mysql_fetch_assoc($result)) {
        echo $row["id"].": ".$row["title"]."<br/>";
    }
}
?>

<form method="get" action="hack.php">
  <input type="text" name="q"/>
  <input type="submit" />
</form>
```

Now create a database called <code>imdb</code> with PHPMyAdmin and execute the following SQL:

```sql
CREATE TABLE IF NOT EXISTS `movies` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) AUTO_INCREMENT=4;

INSERT INTO `movies` (`id`, `title`) VALUES
(1, 'Harry Potter'),
(2, 'Lord of the Rings'),
(3, 'Rise of the Silver Surfer');

CREATE TABLE IF NOT EXISTS `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) NOT NULL,
  `password` varchar(32) NOT NULL,
  `email` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) AUTO_INCREMENT=3 ;

INSERT INTO `users` (`id`, `username`, `password`, `email`) VALUES
(1, 'admin', 'qewrtqwert', 'admin@imdb.com'),
(2, 'user', 'secret', 'mylittlepony@stupid.com');
```

Now go to <a href="http://localhost/hack.php">http://localhost/hack.php</a>. It should look like this:

<figure class="aligncenter">
            <a href="../images/2013/07/hack-screenshot.png"><img src="../images/2013/07/hack-screenshot.png" alt="Screenshot of my minimal example" style="max-width:300px;max-height:85px" class="size-medium wp-image-73971"/></a>
            <figcaption class="text-center">Screenshot of my minimal example</figcaption>
        </figure>

When you search for "Harry Potter" it should show you "1: Harry Potter". Note that there could be a lot of information, but I wanted to keep this example as small as possible.

<figure class="aligncenter">
            <a href="../images/2013/07/hack-normal-use.png"><img src="../images/2013/07/hack-normal-use.png" alt="Normal use of the web service" style="max-width:300px;max-height:77px" class="size-medium"/></a>
            <figcaption class="text-center">Normal use of the web service</figcaption>
        </figure>

This resulted in the following query:

```sql
SELECT * FROM `movies` WHERE title='Harry Potter'
```

But a Hacker could also enter a string like this: <code>' OR '1'='1</code>:

<figure class="aligncenter">
            <a href="../images/2013/07/hack-hacky-use.png"><img src="../images/2013/07/hack-hacky-use.png" alt="What a hacker could do" style="max-width:300px;max-height:94px" class="size-medium"/></a>
            <figcaption class="text-center">What a hacker could do</figcaption>
        </figure>

Even worse, the attacker could know that you use MySQL. Then he might know that MySQL uses <a href="http://dev.mysql.com/doc/refman/5.1/en/information-schema.html">INFORMATION_SCHEMA tables</a>. He might enter this into the title input element:

```sql
' UNION SELECT table_name, table_type FROM information_schema.tables WHERE '1'='1
```

which results in this query:

```sql
SELECT * FROM `movies` WHERE title='' UNION SELECT table_name, table_type FROM information_schema.tables WHERE '1'='1'
```

which gives:

<figure class="aligncenter">
            <a href="../images/2013/07/sql-injection-example.png"><img src="../images/2013/07/sql-injection-example.png" alt="Attacker got all table names" style="max-width:268px;max-height:300px" class="size-medium"/></a>
            <figcaption class="text-center">Attacker got all table names</figcaption>
        </figure>

This way, the attacker gets all table names from all databases on this machine. So he essentially can get everything stored in your database. And, of course, after getting everything he could drop it:

<figure class="aligncenter">
            <a href="../images/2013/07/exploits_of_a_mom.png"><img src="../images/2013/07/exploits_of_a_mom.png" alt="xkcd 327: Exploits of a mom" style="max-width:300px;max-height:92px" class="size-medium"/></a>
            <figcaption class="text-center">xkcd 327: Exploits of a mom</figcaption>
        </figure>

<h2>History</h2>
Just a few famous examples to show you that this happens all the time:

<ul>
  <li>2005: USC admissions page (<a href="http://www.theregister.co.uk/2005/07/06/usc_site_cracked/">source</a>)</li>
  <li>2006: 800,000 datasets of personal information of students of UCLA (<a href="http://www.schneier.com/blog/archives/2006/12/major_privacy_b_1.html">source</a>)</li>
  <li>2008, 2009: Heartland Payment Systems; 130 million credit and debit cards (<a href="http://www.zdnet.com/blog/government/gonzales-just-tip-of-iceberg-in-heartland-attack/5252">source 1</a>, <a href="http://www.computerworld.com.au/article/315418/sql_injection_attacks_led_massive_data_breaches/">source 2</a>, see <a href="http://en.wikipedia.org/wiki/Albert_Gonzalez">Albert Gonzalez</a>)</li>
  <li>2010: Royal Navy Website; Passwords and Usernames stolen (<a href="http://www.eweek.com/c/a/Security/Hacker-Hits-British-Navy-Website-With-SQL-Injection-Attack-108377/">source</a>)</li>
  <li>2011: Sony:

<blockquote>LulzSec says it accessed the passwords, email addresses, home addresses and dates of birth of one million users. The group says it also stole all admin details of Sony Pictures, including passwords. 75,000 music codes and 3.5 million music coupons were also accessed, according to the press release.</blockquote>


(<a href="http://www.thewhir.com/web-hosting-news/hackers-attack-sony-pictures-with-single-sql-injection">source</a>)</li>
  <li>2011: Expedia (<a href="http://www.eweek.com/c/a/Security/Expedias-TripAdvisor-Member-Data-Stolen-in-Possible-SQL-Injection-Attack-522785/">source</a>)</li>
  <li>2011: MySql - Usernames and passwords stolen (<a href="http://www.infoworld.com/d/security/mysql-website-falls-victim-sql-injection-attack-155886">source 1</a>, <a href="http://seclists.org/fulldisclosure/2011/Mar/309">source 2</a>)</li>
  <li>2011: Comodo (<a href="http://www.infosecurity-magazine.com/view/18265/another-comodo-partner-attacked-using-sql-injection/">source 1</a>, <a href="http://www.heise.de/security/meldung/Erneut-Comodo-SSL-Registrar-gehackt-1250208.html">source 2</a>)</li>
</ul>


<h2>Solutions</h2>
<ul>
  <li>Sanitize user input, e.g. with <a href="http://de2.php.net/mysql_real_escape_string">mysql_real_escape_string</a></li>
  <li>Use <a href="http://php.net/manual/en/pdo.prepared-statements.php">prepared statements</a></li>
  <li>Switch of <a href="http://php.net/manual/en/function.error-reporting.php">error reporting</a> (this makes attacks more difficult, but doesn't prevent them)</li>
</ul>

<h2>See also</h2>
<ul>
  <li><a href="../challenge-websites/">Challenge Websites</a>: Try if you can write SQL injections yourself ☺</li>
</ul>
