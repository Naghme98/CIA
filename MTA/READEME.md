In this assignment I tried to make a **Mail Transfer Agents** using **EXIM**:


## Task 1: Install

1. Because it is important that an MTA be correct and secure it is often signed using a digital PGP signature. If your MTA is signed then make sure you have downloaded the correct sources by checking the validity of the key and the signature.
2. There are a number of options that you will have to enter before compilation,so that the functionality can be compiled into the program. Make sure the basic install holds all the necessary functionality. Show the options you configured.
3. Add a local account on your experimental machine and make sure that the MTA can deliver mail to it. Show the required configuration.
4. Add to your log an email received by this account. Do not forget the full headers!
5. Also make sure that any email intended for postmaster@st(X).sne21.ru is delivered to this account. Show the full email as delivered to the new account and the required configuration.

### Answer:

1. For this part, I did the check as it is inside the below picture.

![](https://i.imgur.com/M5ik55i.png)
<p align = "center">
  <i>Figure 1: Check the exim's source code</i>
 </p>
 
2. For this step, I used exim's wiki to do. 

At first we needed to extract the file:
```
tar -zxvf exim-4.94.2.tar.gz
```
Then : 
```
apt-get install libpcre3 libpcre3-dev 
```
To create the changes I needed to create the Makefile:

```
3. cp ../src/EDITME exim/Local/Makefile
```

The first configuration lines are as the following:
I needed to specify the bin, conf and spool and log file and start using the openssl. Then I create the exim user.

```
BIN_DIRECTORY=/usr/exim/bin
CONFIGURE_FILE=/usr/exim/configure
EXIM_USER=exim
LOG_FILE_PATH=/var/log/exim/exim_%slog
SPOOL_DIRECTORY=/var/spool/exim
USE_OPENSSL=yes
TLS_LIBS=-lssl -lcrypto
```

Exim needs DB and for that, I used the following command to install the requierd one.

```
apt-get install libdb-dev
```
After that I ran "make" then "make install "

At the end, I needed to add its bin path to $PATH that made me able to use "exim" command.

```
export PATH=$PATH:/usr/exim/bin/
```
3. I created local user "naghme1" and by the below command first I checked that if exim is working well and then checked its connection to the naghme1 local user.


```
#checking exim
exim -bV

#checking local user
exim -bt naghme1
```

![](https://i.imgur.com/jDXaQOb.png)

![](https://i.imgur.com/ZdmbeoC.png)


<p align = "center">
  <i>Figure 2,3: check exim and local user</i>
</p>
  
For sending a message to this account I did as it is in the following picture. And it is deliverd.

![](https://i.imgur.com/oje9oNG.png)
<p align = "center">
  <i>Figure 4: Sending local email</i>
</p>
Inside /var/mail/naghme1 we can see the received emails for this accout.

![](https://i.imgur.com/cfoex7Q.png)
<p align = "center">
  <i>Figure 5: Received email</i>
</p>  

5. For this part, I needed to change /etc/aliases and add the following part. That means every emails for postmaster will go to naghme1

```
postmaster: naghme1
```

![](https://i.imgur.com/3vVMEA6.png)

![](https://i.imgur.com/JnZG0iu.png)
<p align = "center">
  <i>Figure 6: Test for postmaster</i>
</p>  

## Task 2: Sending mail - email validation - SPF & DKIM


1. Write a small paragraph that highlights the advantages and disadvantages of SPF and DomainKeys Identified Mail (DKIM). What would you choose at a first glance and why?
2. Set up your mail server to use a relay server to be able to send outgoing mail for different domains (contact your TA to obtain address/credentials to the relay server and SPF/DKIM records).
3. Test how your mail is delivered to commonly known mail servers (f.e. gmail). Provide full email/MTA headers to see how SPF/DKIM were delivered.


### Answers:

1. Sender Policy Framework, or SPF is used for email authentication. Beacuse of this authentication, if an attacker tries to send fake email from our domain, the receiving email server sees that it’s from a malicious source, and flags it (attacker cannot make that authentication which is provided by our domain). So, in this way, SPF will Stops phishing attacks. The other advantage is for our domain reputation. By this authentication process we will say that we have protection from attacks so, do not flag mails from us as spam or somthing. One of its disadvantages is about when someone else forwards an email sent from your domain, their IP address won’t be listed on your SPF record. The receiving email server sees this and mistakenly flags it and the email fails SPF. Besides, we can name Limit of 10 DNS lookups for SPF records and Difficulty of maintaining SPF records as other disadvantages.
DKIM (DomainKeys Identified Mail) will add a digital signature to the headers of an email that makes messages remain unaltered in transit between sending and receiving servers. The primary advantage of this system for email recipients is that it offers the signing domain’s ability to reliably record genuine email traffic. This allows domain-based lists to be more effective. This offers ease in detecting some types of phishing attacks. On the other side, A malicious person can write an email from a reputable domain and get this message signed with DKIM and send it to any mailbox where it can be retrieved as an archive and get a signed copy of the email. This signed copy can be forwarded to many recipients without any control. The email provider can block the person who sent the message but will not be able to stop the propagation of the already signed message.



2. For this part, I got my credentials from you (SPF and DKIM) and add them inside my zone file.

![](https://i.imgur.com/TxbFi0C.png)
<p align = "center">
  <i>Figure 7: zone update with spf and DKIM</i>
</p>

After that, I needed to add some additional parts in to my exim configure file and restart it.

```
	
smarthous_add = smtp.eu.mailgun.org
SMART_PORT = 587
SMART_USER = postmaster@st5.sne21.ru
SMART_PASS = bc584dce6a4841d2906ade60ecbb715c-45f7aa85-6f294c33


#uncomment 
daemon_smtp_ports = 25 : 465 : 587

#Inside the router part

.ifdef smarthous_add
smarthost:
  driver = manualroute
  domains = ! +local_domains
  transport = smarthost_smtp
  route_list = * SMARTHOST_ADDR
  host_find_failed = defer
  no_more
.endif

#inside the transport part

.ifdef smarthous_add
smarthost_smtp:
  driver = smtp
  .ifdef SMART_PORT
  port = SMART_PORT
  .endif
  hosts_require_auth = smarthous_add
  hosts_require_tls = smarthous_add
.endif

#inside the authentication part

.ifdef smarthous_add
smarthost_login:                            
  driver = plaintext                    
  public_name = LOGIN           
  hide client_send =:SMART_USER:SMART_PASS
.endif  
```

Finally restart it and try to send a message to my gmail account. From these result we can see steps that passed to create a connection to the mailgun and do the authentications. Also in received message we can see that it has a TLS security that is from the mailgun.

![](https://i.imgur.com/QnCHE9j.png)

<p align = "center">
  <i>Figure 8: Sending an email after adding the mailgun</i>
</p>  

![](https://i.imgur.com/OnRAOk7.png)
<p align = "center">
  <i>Figure 9: Email received after adding the mailgun</i>
</p>

## Task3: Mail backup

1. Be a backup
2. Use a backup


### Answer:
1. To be a backup, we need to add our partner's domain in our relay_domains. For this, I edit the following part inside my /usr/exim/configure.

```
domainlist relay_to_domains = st6.sne21.ru
```
After that I wait to receive my partner's emails.
As I undrestand, these emails are storing inside the mailgun and not my exim server. Cause they are sending back to the mailgun and if his server is not up, I believe they should again come back to me and create and create a loop. As I didn't see they coming back again for me, I think may be I have some misconfigurations (due to what you said inside the group)
In addition, there is no way for me to send and force the emails to send back to their original owner after his server is up. Only maybe if he come back on his server before mine send it back to the mailgun he can receive it.


![](https://i.imgur.com/7syFngD.png)
<p align = "center">
  <i>Figure 10: Emails received as a backup</i>
</p>

2. For this part, I needed to add an MX record pointing to backup server's email address ( in Figure 7 it is presented)

After that, I turned my server off and wait for him to receive my emails.


## Task 4: Mailing loops

1. Create an email loop with your partner from domain to domain using email aliases.
2. Send an email to the loop using your own email address and see what happens on your MTA.
3. Can you change the behaviour of your MTA in response to this loop?


### Answer:

1. For this part, I add a record in my /etc/aliases like this and he add me there in his file too:

```
naghme1: pacman@st6.sne21.ru
```

2. And then, tried to send an email to him but, he didn't receive anything from me although I had a success state. After that I asked him to send mine, and I received it and send it back to him but he got nothing. 
3. I wasn't able to see the loop, so I didn't do anything for this part. But, I had some searches that they said we can handle this, by examining email's headers.

![](https://i.imgur.com/Zfj2Gin.png)
<p align = "center">
  <i>Figure 11: Sending email status</i>
</p>

## Task 5: Virtual Domains

1. Create a new subdomain within your domain and add an MX entry to it.
2. Then extend your MTA configuration to handle virtual domains, and have it handle the email for the newly created domain.
3. Validate that you are now receiving emails for both domains.


### Answers:

1. I create another zone file and add it inside nsd.

![](https://i.imgur.com/6zchkYs.png)
<p align = "center">
  <i>Figure 12: subdomain zone file</i>
</p>  

![](https://i.imgur.com/UpmO5vl.png)
<p align = "center">
  <i>Figure 13: add new zone inside nsd</i>
</p>

2. After that I add these lines to make exim use virtual domains:

![](https://i.imgur.com/lmdWl7k.png)
<p align = "center">
  <i>Figure 14: /etc/exim/configure add after route begin</i>
</p>

And I create the virtual-domain files

3. I tried it but not successful (sorry no time to debug)

![](https://i.imgur.com/dwJX3GN.png)

## Task 6: Transport Encryption

1. Which one is better, SSL/TLS or STARTTLS, why?
2. Which one is actually in useforSMTP?
3. Add transport encryption to your MTA.
4. Eventually force the transport to be encrypted only(refuse non encrypted transport).
5. Proceed with validation (proof or acceptance testing), as usual.


### Answers:

1. STARTTLS is a Channel Security Upgrade for safer delivery of message. It tells an email server that an email client (including an email client running in a web browser) wants to turn an existing insecure connection into a secure one. TLS operates as Application Layer protocols and it is used to create encryption. Not only does SSL/TLS protect user information by encrypting the connection, but it also verifies if the users are connected to the right server. Therefore, anyone who intercepts your encrypted emails will be left with unusable text because only the client and the email server have the keys to decoding the messages. So, I think TLS is more secure to be used.
2. I am not sure but I think MTAs commonly use STARTTLS as their basis with combination of TLS to secure the connection. Generally, SSL/TLS is only used between end-clients and servers. STARTTLS is more commonly used between MTA's to secure inter-server transport.
3. For this part I generate key and cert files for my server and signed them myself and use them as TLS cert in /usr/exim/configure file

```
/usr/bin/openssl req -x509 -sha256 -days 9000 -nodes -newkey rsa:4096 -keyout /etc/exim.key -out /etc/exim.cert
```
```
 # changes inside the config file
 
 tls_advertise_hosts = *
 tls_certificate = /usr/exim/exim.cert
 tls_privatekey = /usr/exim/exim.key
```

4. For this part, I am not sure if I did the right configuration or not, cause I don't know how I need to check it I am on a right way or not. Cause all of us are using mailgun as a relay and it uses TLS encryption I couldn't find how to check the correct rejection for not TLS connections.

```
acl_check_rcpt:
  deny
    condition      = ${if and{{eq{$interface_port}{25}} {eq{$tls_cipher}{}} } }
    message        = All port 25 connections must use TLS

```
5. For this part too, I didn't know how to check. But, I somehow used one online site that said would check the domains and the bellow picture is the result for that.

![](https://i.imgur.com/hYuSNUP.png)
<p align = "center">
  <i>Figure 12: checking the server</i>
</p>


---
## References:
1. [DomainKeys Identified Mail](https://www.saleslovesmarketing.co/glossary/domainkeys-identified-mail)
2. [Advantages and Disadvantages of SPF](https://support.powerdmarc.com/support/solutions/articles/60000664618-advantages-and-disadvantages-of-spf)
3. [DKIM / Authentication Advantages and Disadvantages](https://www.pinpointe.com/blog/dkim-authentication-advantages-and-disadvantages)
4. [Are there any pitfalls to DKIM?](https://serverfault.com/questions/111259/are-there-any-pitfalls-to-dkim)
5. [DKIM: What is it and why is it important?](https://postmarkapp.com/guides/dkim)
6. [Building and installing Exim](https://www.exim.org/exim-html-current/doc/html/spec_html/ch-building_and_installing_exim.html)
7. [How to create a new self-signed keys for TLS](https://help.directadmin.com/item.php?id=245)
8. [Drop non-TLS connection on EXIM](https://serverfault.com/questions/705472/drop-non-tls-connection-on-exim)
9. [Is STARTTLS less safe than TLS/SSL?](https://newbedev.com/is-starttls-less-safe-than-tls-ssl)
10. [What Are SSL, TLS, & STARTTLS Email Encryption?](https://www.sparkpost.com/resources/email-explained/ssl-tls-starttls-encyption/)
11. [SSL, TLS, and STARTTLS Explained in 5 Minutes](https://www.anubisnetworks.com/blog/ssl_and_tls_explained_in_5_minutes)

