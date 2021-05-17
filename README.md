# Prime:1 ~Vulnhub Writeup
Vulnhub writeup for OSCP Exam based machine. 

Author: Suraj

Download link: https://download.vulnhub.com/prime/Prime_Series_Level-1.rar

Find IP of target machine using nmap command - **nmap -sn 192.168.122.0/24**

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled.png)

### SCANNING

  1.  nmap -p-  IP (full port scan)

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%201.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%201.png)

  2.  nmap IP (normal)

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%202.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%202.png)

  3.  nmap -sV --script -vuln -A -p [ports] IP_address

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%203.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%203.png)

### ENUMERATION

PORT 80

1. Check view source - view-source:[http://192.168.122.133/](http://192.168.122.133/)

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%204.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%204.png)

2. Save image file to extract data from image

Tool - 

**a) binwalk   =  binwalk -e [image]**

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%205.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%205.png)

**b) exiftool  =  exiftool [image]**

**c) strings  =  strings [image_name] > [file_name]** 

**tail -** To read last 10 lines of file

**head -** To read first 10 lines

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%206.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%206.png)

We found nothing in image file. 

In nmap vulnerability scanning we found wordpress directory and wp-login.php

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%207.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%207.png)

3. Find user name in site

  We found user name **victor**

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%208.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%208.png)

4. Now enumerate wordpress site using **wpscan**

    command - **wpscan --url 192.168.122.133/worpress --enumerate t** 

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%209.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%209.png)

***IF Nothing found in wpscan. Then try to authenticate using default cred or by bruteforcing.

 5. Bruteforcing directory using **dirb**

**dirb [http://192.168.122.133](http://192.168.122.133) -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt**

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2010.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2010.png)

we found dev directory in dirb

Open [http://192.168.122.133/dev](http://192.168.122.133/dev). Now we are at level 0.

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2011.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2011.png)

**Bruteforcing using gobuster**

**root@kali:~# gobuster dir -u [http://192.168.122.133](http://192.168.122.133/) -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt**

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2012.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2012.png)

we found index.php, image.php and secret.txt file and got some hint in secret.txt

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2013.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2013.png)

***It is saying do some more fuzz on every php page. We have index.php and image.php.

6. Use **WFUZZ** tool to find parameter in index.php and image.php

**root@kali:~# wfuzz -w /usr/share/SecLists/Discovery/Web-Content/common.txt --hc 404 --hw 12 192.168.122.133/index.php?FUZZ=location.txt**

Found file parameter.

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2014.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2014.png)

7. Now Browse 192.168.122.133/index.php?file=location.txt

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2015.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2015.png)

***we got right parameter '**secrettier360' for other php page which image.php**

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2016.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2016.png)

## EXPLOITATION

8. Once you found right parameter **go for LFI**

**192.168.122.133/image.php?secrettier360=../../../../etc/passwd**

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2017.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2017.png)

Now open /home/saket/password.txt, we got password **follow_the_ippsec**

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2018.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2018.png)

9. Until now, we have found following information

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2019.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2019.png)

10.  According to above gathered information, we have Following possibilities:

a. wordpress User - saket and password  - follow_the_ippsec

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2020.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2020.png)

b. SSH user saket and password follow_the_ippsec

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2021.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2021.png)

c. Wordpress user Victor and password follow_the_ippsec

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2022.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2022.png)

Finally we logged in with user victor.

## Taking Reverse shell from wordpress

Search blogs for taking reverse shell from any CMS

1. Explore the appearance tab and go to themes editor.
2. Select themes and search for php files. We found 404.php file in twentynineteen theme but it is not writable. 

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2023.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2023.png)

3.  Explore more php pages and we got secret.php file which is writable.

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2024.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2024.png)

4. Inject your reverse shell php code and save file.

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2025.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2025.png)

5. Start listener on Attacker machine 

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2026.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2026.png)

***If you are getting trouble in finding theme url of any cms, search it on github. Search : github wordpress

6. Open **192.168.122.133/wordpress/wp-content/themes/twentynineteen/secret.php**

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2027.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2027.png)

![Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2028.png](Prime%201%20machine%20Writeup%20de9a733e9341441f87863c265eeca84a/Untitled%2028.png)

Finally we got reverse shell of target machine.
