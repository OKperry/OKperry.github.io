---
layout: post
title:  "THM - Agent Sudo Write Up"
---


### Enumaration

In this first stage we will scan the server for any open ports. This allows us to find vulnrabale services that can be later exploited. I will use the command - nmap -sV -p- [MACHINE IP], this gives us a scan of the common ports (-p-) as well as information about the services running  (-sV). This is our result

![image-20210821092835904](/assets/thm-agent-wu/image-20210821092835904.png)

We now know the answer to the first question -

Q: How many ports are open? A: 3

In addition, from the nmap scan we can now see that the server is running a http service(website). On naviagtion to http://[machineIP]:80 we see the following - 
![image-20210821094751547](/assets/thm-agent-wu/image-20210821094751547.png)

As seen in the screenshot to properly access the site we need to set our user agent to our code name.

Q: How do you redirect yourself to a secret page? A: user-agent

Judging from the format used in this web page the code name is your first  letter of your name. At this point we could manually change our user  agent in the browser settings, install an extension or ,what I will be using, Burpsuit. In burpsuit I will be doing the following; turn on intercept and load the page, we will be given the following -

![image-20210821100525087](/assets/thm-agent-wu/image-20210821100525087.png)

Instead of manually changing the user agent and having to try each letter of  the alphabet I will send this request to the intruder, this allows us to select a payload for the user agent and try each entry in the payload.

![image-20210821100845932](/assets/thm-agent-wu/image-20210821100845932.png)

![image-20210821100905200](/assets/thm-agent-wu/image-20210821100905200.png)

This now allows us to run the intruder, and it will enter each letter into the user agent field (indicated by the two symbols) and give us the HTTP status code.

![image-20210821101138870](/assets/thm-agent-wu/image-20210821101138870.png)

As you can see in the screenshot above the status for user-agent set as C  gives us a different status from every other request besides R. If we  view this request in the browser we will be redirected to the new page -

![image-20210821101551542](/assets/thm-agent-wu/image-20210821101551542.png)

Q: What is the agents name? A: chris

### Hash cracking and brute-forcing

First off, from our initial nmap scan we are aware of the fact that the server is running a ftp service, now we are aware of a possible user name we can try bruteforcing the password for the user "chris". We will use hyrda to bruteforce the ftp login with the following command -

![image-20210821102239741](/assets/thm-agent-wu/image-20210821102239741.png)

Q: What is the FTP password? A: crystal.

"-l" this lowercase l specifies that we only want to use one username, the capital -P specifies that we want to attemp multiple passwords from the file we choose.

Now we can connect to the ftp server. Ftp -p [ip] [port] - ftp -p 10.10.224.138 21. Once we are connected we will see the following - 
![image-20210821122744945](/assets/thm-agent-wu/image-20210821122744945.png)

Now I will use the "get" command to make a local copy of each file to perform further analyis. If we print the txt file we will see the following -

![image-20210821123249537](/assets/thm-agent-wu/image-20210821123249537.png)

I first tried to use steghide on the images, but I needed a passphrase. To try to get something more out of the files I ran binwalker which looks for embeded files or executables.

![image-20210821124232723](/assets/thm-agent-wu/image-20210821124232723.png)

On extraction of the file we can see that there is a hidden zip inside the PNG. However, this zip is protected with a passcode, to crack a zip file we will use zip2john then john.

![image-20210821124540133](/assets/thm-agent-wu/image-20210821124540133.png)

Now we can open the zip using the password found from the bruteforce. From the ZIP we now have a txt file with the following information -

![image-20210821124726556](/assets/thm-agent-wu/image-20210821124726556.png)

I tried to use this string "QXJlYTUx" as the steghide passphrase, but it did not work, maybe its encoded? To check I will use cyberchefs magic feature -

![image-20210821125213850](/assets/thm-agent-wu/image-20210821125213850.png)

Now I will check if "Area51" works using stegohide -

![image-20210821132319637](/assets/thm-agent-wu/image-20210821132319637.png)

![image-20210821132332145](/assets/thm-agent-wu/image-20210821132332145.png)

Now we can see that Agent J is called james and his password is "hackerrules!"

From all of that we were able to get the <u>FTP password, ZIP password, steg password, agents name and ssh password</u>. In addtion we can now connect to james machine via ssh to gain the user flag.

As for the random second question regarding the JPG image in the users' directory if we download the image using scp and then do a reverse image search(I used TinEye) it will need us to the following article -

![image-20210821140327700](/assets/thm-agent-wu/image-20210821140327700.png)

### Privalege escalation

In order to escalate the user to a root I will first change directory to tmp, so I can run scripts from there. To get the scripts into the tmp file I will use scp - ![image-20210821135646664](/assets/thm-agent-wu/image-20210821135646664.png)

The script I will be running is linpeas which is great for finding possible ways of priv escaltion.

![image-20210821135733127](/assets/thm-agent-wu/image-20210821135733127.png)

From this script we can see the following(linpeas automitcally highlights the most likely methods of priv escalation). After a quick google search we can find the following -

![image-20210821135841931](/assets/thm-agent-wu/image-20210821135841931.png)

![image-20210821135911569](/assets/thm-agent-wu/image-20210821135911569.png)

This exploit includes a python script, we will use scp again to get this on the machine -

![image-20210821135955210](/assets/thm-agent-wu/image-20210821135955210.png)

![image-20210821140122617](/assets/thm-agent-wu/image-20210821140122617.png)

And just like that we have gained root access and can now get the root flag.



### Evaluation

This CTF did a good job at demonstrating that security through obscurity can still be vulnerable. It did a good job at showing the importance of using secure and recognized protocols for communication instead of making your own bootleg communication systems via weird HTTP websites(doubt many people do this). The implementation ofsteganography made the CTF slighltly more interesting. Overall, this CTF did a good job of demonstrating some basic pentesting techniques with a tiny bit of hand holding. 



