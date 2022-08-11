# Introduction
This lab has two sections. The first section covers web servers and the second section covers
directory services

I am going to use **Nginx**:

## Task 1: Install & Configure Virtual Hosts
1. Fetch, verify, build and install the webserver daemon from the source.
Note: Some features of your web server may be built-in or modularized. Enable at least SSL/TLS during your installation.

2. Define the root directory and then two virtual hosts (and configure DNS records or wildcard accordingly):

3. Create a simple, unique HTML page for each virtual host to make sure that the server can correctly serve it.

4. Check the configuration syntax 1, start the daemon and enable it at boot time.


5. Use curl to display the contents of a full HTTP/1.1 session served by your server.


6. Explain the meaning of each request and reply header.

### Answers:

1. 
* Download source from Nginx site:

```
wget  http://nginx.org/download/nginx-1.20.0.tar.gz
```
* Verify downloaded tar file.

```
gpg --verify nginx-1.20.0.tar.gz.asc nginx-1.20.0.tar.gz
```
However, the gpg command failed to check the signature as I didn’t have the author’s public key 520A9993A1C052F8 in my server (I forgot to take a picture from this error). Hence, I needed to grab the public key from a key server (such as pgpkeys.mit.edu) or download it from the author’s website. After searching inside the Nginx site I couldn't find anything appropriate (I downloaded some keys, but they were not the wanted ones.)

I received the key with the following command:
```
gpg --keyserver pgpkeys.mit.edu --recv-key 520A9993A1C052F8
```
Then I was able to verify it (Figure 2)

![](https://i.imgur.com/2dOBN4Y.png)
<p align=center>
  <i>Figure 1: Download public key</i>
</p>

![](https://i.imgur.com/fkoPWYJ.png)
<p align=center>
  <i>Figure 2: Varify tar file</i>
</p>

* Extracting tar file

```
tar -zxvf nginx-1.20.0.tar.gz
```
* After going inside the Nginx directory I needed to configure Nginx with some features (These should be on a line, but for clarity, I pasted each one in different lines). The first eight feature lines indicate default paths for the bin, log, etc. After that, some other features are listed like enabling the SSL module.



```
./configure 
--prefix=/var/www/html 
--sbin-path=/usr/sbin/nginx 
--conf-path=/etc/nginx/nginx.conf 
--http-log-path=/var/log/nginx/access.log 
--error-log-path=/var/log/nginx/error.log 
--lock-path=/var/lock/nginx.lock 
--pid-path=/var/run/nginx.pid 
--modules-path=/etc/nginx/modules 
--with-pcre 
--with-http_ssl_module 
--with-http_image_filter_module=dynamic 
--with-http_v2_module 
--with-stream=dynamic
--with-http_addition_module 
--with-http_mp4_module
```
* Next would be: make and make install


![](https://i.imgur.com/LFENIIW.png)
<p align=center>
  <i>Figure 3: Installed Nginx</i>
</p>

* I didn't know we can add a source file downloaded package inside system service, but I saw an instruction on the internet, and after following it, I was able to use e.g "systemctl restart Nginx"
The steps are as following:

```
nano /lib/systemd/system/nginx.service
```
I changed the file as like as the following picture. It just specifies the commands for staring or reloading, etc.

![](https://i.imgur.com/bwLfvwd.png)
<p align=center>
  <i>Figure 4: nginx.service</i>
</p>

```
systemctl restart nginx
```

![](https://i.imgur.com/AXGDZmJ.png)
<p align=center>
  <i>Figure 5: Nginx status</i>
</p>

* Allow http inside ufw:
```
ufw allow 'Nginx HTTP'
```
![](https://i.imgur.com/ds5wmAW.png)
<p align=center>
  <i>Figure 6: UFW allow http</i>
</p>


2. AND 3. For two virtual hosts, I didn't pay attention that you mentioned using aaa and bbb specifically, and I create my virtual hosts with the name of "site1 and site2"

* Nginx itself has a default server block (/var/www/html), but now we need to create two server blocks for our virtual hosts.

```
mkdir -p /var/www/site1.st5.sne21.ru/html
mkdir -p /var/www/site2.st5.sne21.ru/html

```
-p is telling mkdir to create needed parent directories.

* Change DNS to serve these two virtual hosts (site3 is for task 3 cause I forgot to take a picture before doing task 3). I added two CNAME for being able to connect to www.site1 and two A records for site1 and site2:

![](https://i.imgur.com/IIetEiI.png)

<p align=center>
  <i>Figure 7: DNS adaption</i>
</p>


* Create HTML file for site1 and site2:

```
nano /var/www/site1.st5.sne21.ru/html/index.html
```

![](https://i.imgur.com/p2lofCq.png)

![](https://i.imgur.com/OmDXr7F.png)

<p align=center>
  <i>Figure 8: HTML file for both virtual hosts</i>
</p>

* Now is time for creating server blocks for Nginx to serve our hosts. As I said before, it has its default, and for creating a new one, I will copy the default and change the file.

```
cp /etc/nginx/sites-available/default /etc/nginx/sites-available/site1.st5.sne21.ru
```

* In that file, I set that they will use port 80 to listen on it and specify the root file address (/var/www/site1.st5.sne21.ru/html). Then the server name and an alias for it (www.site1) Same process goes on for the site2.

![](https://i.imgur.com/QSxEIAd.png)
![](https://i.imgur.com/5rNqnZ5.png)

<p align=center>
  <i>Figure 9: Server block for site1</i>
</p>

* Now that I have my server block files, I need to enable them by creating symbolic links from these files to the sites-enabled directory, which Nginx reads from during startup.

```
ln -s /etc/nginx/sites-available/site1.st5.sne21.ru /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/site2.st5.sne21.ru /etc/nginx/sites-enabled/
```

4. For checking the syntax:

```
nginx -t
```

![](https://i.imgur.com/G3DTC20.png)

<p align=center>
  <i>Figure 10: Nginx syntanx checking</i>
</p>

* restart nginx:

```
systemctl restart nginx
```

* enble it at boot time:
```
systemctl enable nginx
```

5. To show the contents of a full HTTP/1.1 session with Curl:

```
curl -v site1.st5.sne21.ru
```

![](https://i.imgur.com/j3TZCnb.png)
<p align=center>
  <i>Figure 11: curl site1 full HTTP session</i>
</p>
  
![](https://i.imgur.com/vtb004W.png)
<p align=center>
  <i>Figure 12: Opening site1.st5.sne21.ru in the browser</i>
</p>

6. Meaning of request and reply headers:
    * GET / HTTP/1.1: This will request resources(default file in root cause I didn't set anything special for example :/file.txt) from the server and HTTP 1.1 is the latest version of Hypertext Transfer Protocol.
    *  Host: site1.st5.sne21.ru: Hostname I am requesting for.
    *  User-Agent: curl/7.64.1: It will send the user agent's name and information that is sending the request for that page. Here I am using Curl if I was using firefox, it would be firefox.
    *  Accept: */*: simply means that any data of whatever mime-type is accepted and the server may choose what to return to the requesting client.
    *  HTTP/1.1 200 OK: Indicates that the request has succeeded. The resource has been fetched and is transmitted in the message body
    *  Server: nginx/1.20.0: Web server name and version
    *  Date: the date that request was created and responded to.
    *  Content-Type: text/html: The text/html content type is an Internet Media Type. That says the server is sending text this is in the format of html content for us. If it was an image, it would be image/png.
    *  Content-Length: Length of the response content.
    *  Last-Modified: Contains the date and time at which the origin server believes the resource was last modified.
    *  ETag: The ETag HTTP response header is a qualifier for determining the version of a web page resource from a website’s server that was requested by the requestor.ETag is used for efficient browser cache in order to decrease bandwidth consumption. If a website uses ETag for its web page resources, the server won’t need to send a full response for every request if the resources don’t change. Lastly, Etags prevents “mid-air collisions” or “simultaneous updates of resources”.
    *  Accept-Ranges: Indicates that bytes can be used as units to define a range





## Task 2: SSL/TLS

1. Enable SSL/TLS and tune the various settings to make it as secure as possible
2. Describe how you created your own certificate(s) e.g. with Let’s encrypt (certbot) or self-signed and re-validate every virtual-host . Explain your security tuning process.


### Answers:

For enabling SSL, we need a certificate. For this certificate, we can create with Openssl and self-sign it or get a certificate from CA. Let’s Encrypt is a Certificate Authority (CA) that I will use Certbot to obtain a free SSL certificate for Nginx on Ubuntu.

*  add the repository to get Certbot from it:

```
add-apt-repository ppa:certbot/certbot
```
* Update package list:

```
apt-get update
```
* install Certbot’s Nginx package with apt-get

```
apt-get install python-certbot-nginx
```
* Allow HTTPS in firewall (enabling full mode for nginx):

```
ufw allow 'Nginx Full'
ufw delete allow 'Nginx HTTP'
```


![](https://i.imgur.com/QVJo6JZ.png)

<p align=center>
  <i>Figure 13: Ufw update to allow https</i>
</p>

* Obtaining an SSL Certificate:
The Nginx plugin will take care of reconfiguring Nginx and reloading the config whenever necessary (For example, by running the following command, changes will be applied inside server blocks)

```
certbot --nginx -d site1.st5.sne21.ru -d www.site1.st5.sne21.ru

certbot --nginx -d site2.st5.sne21.ru -d www.site2.st5.sne21.ru
```
By this command, we will tell the certbot for which host we are requesting a certificate. Then it will ask us for an email and whether we want to redirect all HTTP requests to HTTPS or not. Finally, it will create our certificate and print the directory of it.

In figure 15, you can see that there is no listening part on port 80 that we had before, and at the end of the server block, we have some information there that was added there to listen to 443, and the address of the SSL certificate and key is specified.

Figure 16 shows that site2 has a certificate, and our connection is secure now.

Figure 17 shows that there is a TLS handshake and confirms SSL connection. Then there is server certificate information, and after that, the request responses to get the content.


![](https://i.imgur.com/rZszi6f.png)
<p align=center>
  <i>Figure 14: certbot download SSL certificate</i>
</p>  


![](https://i.imgur.com/MJfiGCu.png)
<p align=center>
  <i>Figure 15: Changes into server blocks after downloading Certificate</i>
</p>
  
![](https://i.imgur.com/7KSWbw5.png)
<p align=center>
  <i>Figure 16: Test SSL certificate for site2 in browser</i>
</p>  

![](https://i.imgur.com/FfSFmhZ.png)

<p align=center>
  <i>Figure 17: Test secure connection for site1 with curl</i>
</p>

* Check TLS/SSL : using https://www.ssllabs.com/ssltest/ I did the check but, there was a problem and I got B. So, I changed the TLSv in nginx.conf to support v1.3.

![](https://i.imgur.com/5N1jcbM.png)

<p align=center>
  <i>Figure 18: Test SSL server after the change</i>
</p>

## Task 3: Choose one of the options from the following

**Web server security:** Create a new virtual host and a HTML page with content (i.e Administrative area). Enable basic authentication in your web server for your new virtual host. Use password file creation utility and create two users. Verify your work by authenticating against your webpage.



### Answers:

HTTP provides a general framework for access control and authentication. 


![](https://i.imgur.com/hHgPu0w.png)
<p align=center>
  <i>Figure 19: Basic authentication process</i>
</p>

For this section I created www.site3.st5.sne21.ru and add the essential records into the DNS server.


* First creating 2 users (passwords I mean the passwords that htpasswd will ask to set):
    * name: user1 pass: user1
    * name: user2 pass: user2

*  Download Apache util: We need htpasswd to create and generate an encrypted for the user using Basic Authentication. 

```
apt-get install apache2-utils
```

* Create a .htpasswd file (-c is for creating a file):

```
htpasswd -c /etc/nginx/.htpasswd user1
htpasswd /etc/nginx/.htpasswd user2
```

![](https://i.imgur.com/3iT7qS7.png)
<p align=center>
  <i>Figure 20: .htpasswd file </i>
</p>

* Update Nginx configuration: I add the two lines in Figure 20 into site3 server block

* 


![](https://i.imgur.com/mR1uJkA.png)
<p align=center>
  <i>Figure 21: changes into site3 server block to use basic authentication </i>
</p>
  
![](https://i.imgur.com/8DpjpQt.png)
<p align=center>
  <i>Figure 22: Confirm the basic authentication</i>
</p>



# Directory Server:
In this section, you will learn how to deploy and configure a directory service. More precisely
deploying your server with one client.

I am going to use **FreeIPA**.

## Task 1: FreeIPA Directory Server installation and configuration

1. Get familiar with your directory server. What features and capabilities? (very short)
2. Prepare the environment, create/use two VMs (i.e server and client)
3. Install the directory server. you DON’T need to install the directory server from the
source. (e.g. use apt-get).
Hint: you can add FQDN for your server in the /etc/hosts file to make it resolvable, in
case you don’t have access to your previews lab or DNS server.
4. (For FreeIPA) get a Kerberos ticket for admin and list it. Check the admin account exists
in the FreeIPA server.
5. Access/log-in to the web dashboard of FreeIPA server. Create a test user (e.g. your
name) for the next task.

### Answer:

I am going to install it using Docker container:

```
git clone https://github.com/freeipa/freeipa-container.git
cd freeipa-container
```

![image](https://user-images.githubusercontent.com/45916098/184148252-321d3eed-9f62-4fc0-80ff-3365cbe896c8.png)

Now the image is downloading:

![image](https://user-images.githubusercontent.com/45916098/184151473-e59cf09a-34fc-4fab-8ff3-886aa12e73c2.png)

Create a data directory for persistent volume of the FreeIPA container. We shall then mount the volume at /data path of the container.

```
sudo mkdir -p /var/lib/ipa-data
```


## Task 2: Configure (FreeIPA) client

1. Install the client package using a package manager.
2. Check both client-server can ping using their FQDN.
3. Authenticate FreeIPA client to server and verify using the test user that you created in
the preview task.







---
## References:

1. [UNIX / Linux PGP TarBall File Signature Keys Verification](https://www.cyberciti.biz/faq/pgp-tarball-file-signature-keys-verification/)
2. [How to Build NGINX from Source on Ubuntu ](https://www.alibabacloud.com/blog/how-to-build-nginx-from-source-on-ubuntu-20-04-lts_597793)
3. [How To Set Up Nginx Server Blocks (Virtual Hosts)](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-16-04)
4. [HTTP headers | Content-Type](https://www.geeksforgeeks.org/http-headers-content-type/)
5. [Content-Type: image](https://docs.microsoft.com/en-us/previous-versions/office/developer/exchange-server-2010/aa494200(v=exchg.140))
6. [Last-Modified](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Last-Modified)
7. [What is HTTP Etag](https://www.holisticseo.digital/pagespeed/etag/)
8. [How To Set Up Let's Encrypt with Nginx Server Blocks](https://www.digitalocean.com/community/tutorials/how-to-set-up-let-s-encrypt-with-nginx-server-blocks-on-ubuntu-16-04)
9. [How To Configure Nginx to use TLS 1.2 / 1.3 only](https://www.cyberciti.biz/faq/configure-nginx-to-use-only-tls-1-2-and-1-3/)
10. [HTTP authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)
11.  [How To Set Up HTTP Authentication With Nginx ](https://www.digitalocean.com/community/tutorials/how-to-set-up-http-authentication-with-nginx-on-ubuntu-12-10)
12. [How to Enable Basic Authentication on NGINX](https://tecadmin.net/enable-basic-authentication-on-nginx/)
13. [How To Create a Self-Signed SSL Certificate for Nginx in Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-16-04)
14. [Run FreeIPA Server in Docker](https://computingforgeeks.com/run-freeipa-server-in-docker-podman-containers/)
