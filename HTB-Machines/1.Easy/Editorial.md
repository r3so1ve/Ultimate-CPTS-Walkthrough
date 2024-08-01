First we have our IP address 10.10.11.20 so we start from nmap scan:
```
sudo nmap 10.10.11.20 -sCVS -p- -v -T 4 -Pn -oN nmap
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0d:ed:b2:9c:e2:53:fb:d4:c8:c1:19:6e:75:80:d8:64 (ECDSA)
|_  256 0f:b9:a7:51:0e:00:d5:7b:5b:7c:5f:bf:2b:ed:53:a0 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://editorial.htb
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
So we have only port 80 and 22 open, so lets start enumerating
Then after receiving IP address I usually paste it to google to see if the we have 80 port enables, and indeed we need to add it to /etc/hosts file
<!--⚠️Imgur upload failed, check dev console-->
![[Attachments/Pasted image 20240801103022.png]]

```
sudo nano /etc/hosts
```
<!--⚠️Imgur upload failed, check dev console-->
![[Attachments/Pasted image 20240801103226.png]]
After adding the ip and domain name to hosts file I can see the content of the webpage:
<!--⚠️Imgur upload failed, check dev console-->
![[Attachments/Pasted image 20240801103337.png]]

Then I decided to enumerate the target and ran the ffuf to search for directories:
```
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://editorial.htb/FUZZ 
```

The ffuf scan gave me that there is 'upload' page
<!--⚠️Imgur upload failed, check dev console-->
![[Attachments/Pasted image 20240801105351.png]]

So we have file upload functionality.
So basically I tried to upload a php reverse shell, but that does not seem to be working, so I decided to use url section to retrieve some info about the target
<!--⚠️Imgur upload failed, check dev console-->
![[Attachments/Pasted image 20240801111101.png]]
This is SSRF attack and it can usually appear on 5000 port. Now first you need ot paste the url and click the preview button, and after that refresh the intercepted page and get the burp result:
<!--⚠️Imgur upload failed, check dev console-->
![[Attachments/Pasted image 20240801112359.png]]

So we see that we have the path, so now lets see the contend of GET request
<!--⚠️Imgur upload failed, check dev console-->
![[Attachments/Pasted image 20240801112432.png]]
By looking at the response I think that  /api/latest/metadata/messages/authors is interesting so lets see what it returns
<!--⚠️Imgur upload failed, check dev console-->
![[Attachments/Pasted image 20240801112739.png]]
So we found some credentials:
```
username: dev
password: dev080217_devAPI!@
```
<!--⚠️Imgur upload failed, check dev console-->
![[Attachments/Pasted image 20240801113110.png]]

So let's try the ssh with that.
<!--⚠️Imgur upload failed, check dev console-->
![[Attachments/Pasted image 20240801113305.png]]
<!--⚠️Imgur upload failed, check dev console-->
![[Attachments/Pasted image 20240801113325.png]]
And it worked!
Now let's enumerate the directories
And immediately we found user flag
<!--⚠️Imgur upload failed, check dev console-->
![[Attachments/Pasted image 20240801113419.png]]

```
user.txt = b5764f51d0751ea57d039a60332e14bc
```

After some enumeration I found that there is '.git' folder in 'apps' directory and another user - prod
<!--⚠️Imgur upload failed, check dev console-->
![[Attachments/Pasted image 20240801113708.png]]