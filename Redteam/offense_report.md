# Red Team: Summary of Operations

## **Target 1**
  - Planning and Reconnaissance and Vulnerability Identification Phase
  - The exploitation phase
  - Privilege Escalation

## Planning and Reconnaissance and Vulnerability Identification Phase

First we start off by running an nmap scan `nmap -sS -sC -sV -Pn 192.168.1.0/24/` aganist the whole network of the system to identify all the Ip's and what they are doing on the network overrall. 
| HOSTNAME       |IPS            | OPEN PORTS |
|----------------|---------------|------------|
| HOST           | 192.168.1.1   |            |
| KALI (root)    | 192.168.1.90  |            |
| ELK            | 1291.68.1.100 | 22, 9200   |
| TARGET1        | 192.168.1.110 |22, 80, 111, 53625, 53854|


Now that we've identified the target machine `192.168.1.110` as the target WordPress server, and we can start enumerating  with a common wordlist provided by the Wordpress scan `WPSCAN`, using `wpscan --url http://192.168.1.110/wordpress/ `,

[wordpress_results.png]

The results of the scan indicate that the are 3 different wordpress directories and 2 different users found on the system, `steven` and `michael`, Since port 22 is open we can try sshing into the target machine using the following 
command `ssh michael@192.168.1.115` and trying to guess his password to gain access


## The exploitation phase
NOw that we have access to the machine, we can now tranverse throughtout the machine looking the infomation we are trying to obatin.

*flag1*
found in `/var/wwwa/html/service.html`
![image](https://user-images.githubusercontent.com/86163817/131773722-f0adfe46-e450-444e-93d5-2f8347cf783b.png)

Someone left the MYSQL logins in the directory `/var/www/hmtl/wp-config.php`
![image](https://user-images.githubusercontent.com/86163817/131773816-0c367f07-90af-4cb7-b958-994bec8ce977.png)

*flag2*
found in `/var/www/`

![image](https://user-images.githubusercontent.com/86163817/131773834-08b54074-2997-4911-a120-5503fd3b8d85.png)


Both of these flags take some time to find but they weren't that diffcult at all. they're located in similar directories that would  be of interest, While looking 0also looking for the SQL database

Going into the SQL database we can see that the credentials are `root:R@v3nSecurity` taking a look with `mysql -u root -p wordpress` we can see the following databases, the one of interest seems to be `wordpress`



![image](https://user-images.githubusercontent.com/86163817/131773781-f066a986-1490-4e7f-ab0f-edd209660a4e.png)

*flag3* found found `wordpress/wp_posts` in the SQL `wordpress` database

![image](https://user-images.githubusercontent.com/86163817/131773648-ad484222-0144-4f7c-8833-c7be7d477ff8.png)

To get the password for user `steven`, we go into the `wordpress` data and select the `wp_users` and you will find both michael and stevens hashs in the table
![image](https://user-images.githubusercontent.com/86163817/131773676-20d4f752-0154-46c0-bd43-fc8727d3f04d.png)


### Privilege Escalation
To Start off Privilege Esclation we do sudo -l, which  shows us that all of the python library can be used as a sudoer without being able to run sudo.

which gives us a really easy way to escalate our privlege by running the python command `"sudo python -c 'import pty;pty.spawn{"/bin/bash")'`. which will run open a shell that has root privileges
![image](https://user-images.githubusercontent.com/86163817/131773577-172d4240-8c01-499c-b317-fd6d631670db.png)


Flag 4 is found in `/root/`

![image](https://user-images.githubusercontent.com/86163817/131773596-701d7753-6169-4e5b-950e-028ac1cf1452.png)

thats the last flag and all there really is to it.

