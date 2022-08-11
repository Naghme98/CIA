# DNS using Unbound + NSD

## Task 1: Downloading and Installing a Caching Name Server

1. Why is it wise to verify your download?
2. Download the BIND tarball (also if you are doing the Unbound+NSD part) and check its validity using one of the signatures.
3. Which mechanism is the best one to use (signatures or hashes)? Why?
4. make sure your installation does not contain a previous version of the servers, as that can really mess things up (show how to check)
5. Make sure each server will look for its configuration files in Unbound - /usr/local/etc/unbound
6. What is the difference between /etc, /usr/etc, /usr/local/etc
--------------

### Answers:

1. Using hash values and Certificates both are effective. For example in Hashes, any change in the data will result in a different hash value . So, it will ensure us that a program has not been tampered with or corrupted, and we will understand the program you have downloaded is the same as the one published by its developer.


2. For this question I used theses commands (Figure 1).


![Alt text](https://i.imgur.com/39nZgx1.png)
<p align = "center">
   <i>Figure 1: Bind signature validation</i>
</p>

3. Hashes will give us Integrity but not authentication. That is, they are useful for ensuring the file or program you have matches the source, but they provide no way of verifying that the source is legitimate. Also, some mathematical weaknesses make them vulnerable.But,signatures are done with the private key, verification with the public key.  As a result, I believe that Signatures are more secure, but hashes are easier to use. 


4. I am not sure if I got it right or not, but I used these commands to check whether there is another version or not (p.s. there were nothing installed).



![Alt text](https://i.imgur.com/yLCRdwz.png)

<p align = "center">
   <i>Figure 2: Checking Bind and Unbound previous versions.</i>
</p>

5. Whenever I ran this command, it went to that directory and read the unbound.conf file

```
unbound-checkconf
```

6. /usr/local is usually for applications built from source not using common package tools (like apt, ..) . The /etc directory contains system-wide configuration files,

-----------------------------------------------------------
## Task 2,3: Configuring Caching Name Server, Running Caching Name Server


1. Why are caching-only name servers still useful?
2. Installing and Configuration:
    * Root Servers
    * Resolving localhost
    * Access control
    * allow remote control

3. What other commands/functions does unbound-control provide? 
4. What is the difference between stop -> start and reload?

-------------------------------------------------------------

### Answers:

1. Using caching-only name servers speeds DNS queries by building a DNS request cache and will reduces the overhead of zone transfers between name servers on a network. Also caching-nameservers allow resolving private zones


2. After downloading and installing (./configure, make, make install), I went through some steps. But, I will put the final version of the changes:

For having enabling unbound-control I had to run the comment in Figure 3 and add a part to the end of unbound.conf file.

I needed to add my DNS cache as my nameserver inside resolv.conf file to make it use mine (figure 4)

I faced a problem that port 53 was in used by resolved.conf and I needed to solve the problem so, I made a change to the /etc/systemd/resolved.conf  (Figure 5). After that, I created a link through two files (Figure 6)

The final configuration for unbound.conf is in Figure 7. As you can see I set the access-control to my localhost. That means, no one from outside would be allowed to send a request to my cache-only DNS and it would consider requests from localhost only.

Diging result I in Figure 8. It is obvious that for the first query it spent time to get the results but for the second time, the IP address for the wanted destination was ready inside the DNS cache. So time got less.



![](https://i.imgur.com/wZqyx1H.png)
<p align = "center">
   <i>Figure 3: Unbound control setup</i>
</p>
   

![](https://i.imgur.com/pPQCMwC.png)
<p align = "center">
   <i>Figure 4: /etc/systemd/resolved.conf</i>
</p>

![](https://i.imgur.com/sp8CFa4.png)
<p align = "center">
   <i>Figure 5: Resolved.conf</i>
</p>

![](https://i.imgur.com/cBt1Wsz.png)
<p align = "center">
   <i>Figure 6: Linking</i>
 </p>  

![](https://i.imgur.com/qiuT3GF.png)
<p align = "center">
   <i>Figure 7: unbound.conf finall edition</i>
</p>


![](https://i.imgur.com/2STkonO.png)
<p align = "center">
   <i>Figure 8: Dig results after deploying DNS cache</i>
</p>   

![](https://i.imgur.com/uEUWDRs.png)
<p align = "center">
   <i>Figure 9: Dns cache dump</i>
</p>   

![](https://i.imgur.com/TSCc4o7.png)
<p align = "center">
   <i>Figure 10: ubound log file</i>
</p>   


3. With unbound-control, we can stop, start, reload the unbound server. Additionaly, by 'stats' we can see the statistics. Dump_cache is also a useful command and so on.
4. By stoping the server, the server daemon will exits and 'Start' will start the server and will search for configuration file and use it. Reload flushes the cache and reads  the  config file fresh.


* For the next part, I switched to a server with IP address 23.88.50.58 and tried the above process again.

---------------------------------------------------
## Task 4:Authoritative Name Server
1. NSD Configuration
2. What is a private DNS zone? Is stX.sne21.ru private?
3. What information was needed by TAs so they can implement the delegation?
4. Create a forward mapping zone file 
----------------------------------------------------

### Answers:

1. I went through steps to be able to configure it and connect it to the Unbound. So, unbound.conf had alittle changes (figure 11) to add the zone name and these things.
2. A private zone contains information about how to map a domain name and its subdomains used within one or more VPCs to private IP addresses (such as 192.168...) So, I don't think that stX.sne21.ru is a private zone. It was said in the project explanation that we need to set public IP addresses so, it would be accessable from outside. 
3. I think they need to add an NS record and a glue (A record) pointing to my server

TA reply:
> in your case it could be public (since you are running it on a cloud server), but you made private running nsd only available on localhost - so it is private (only available for the machine user and unbound clients)
> Just to clarify, it should be 3(2) things: name of your subdomain, names of nameservers, and IP addresses of this nameservers (since in this case nameserver's name is inside the delegated domain)


![](https://i.imgur.com/lIr9UoG.png)
<p align = "center">
   <i>Figure 11: unbound.conf edited version to work with NSD</i>
</p>   

![](https://i.imgur.com/7AVNvBx.png)
<p align = "center">
   <i>Figure 12: NSD configuration</i>
</p>   

![](https://i.imgur.com/xwPhNlJ.png)
<p align = "center">
   <i>Figure 13: st5.sne21.ru. zone file</i>
</p>   

![](https://i.imgur.com/AZ6ibfX.png)
<p align = "center">
   <i>Figure 14: Dig from local</i>
</p>   

![](https://i.imgur.com/079cu2X.png)
<p align = "center">
   <i>Figure 15: Dig from outside</i>
</p>   

---
## References:

1. [Hash and Signature](https://proprivacy.com/guides/how-why-and-when-you-should-hash-check)
2. [Simple DNS Administration with NSD & Unbound](https://www.pbdigital.org/post/2020-08-31-nlnetlabs-nsd-unbound-omnios/)
3. [OpenBSD DNS Server with unbound and nsd](https://jamsek.dev/posts/2019/Jul/28/openbsd-dns-server-with-unbound-and-nsd/)
4. [An Introduction to DNS Terminology, Components, and Concepts](https://www.digitalocean.com/community/tutorials/an-introduction-to-dns-terminology-components-and-concepts)
5. [How To Use NSD, an Authoritative-Only DNS Server](https://www.digitalocean.com/community/tutorials/how-to-use-nsd-an-authoritative-only-dns-server-on-ubuntu-14-04)
6. [HOWTO - Configure Sub-domains](http://www.zytrax.com/books/dns/ch9/subdomain.html)
7. [NSD](https://wiki.archlinux.org/title/NSD)
8. [Unbound](https://wiki.archlinux.org/title/unbound)
9. [NSD-official documents](https://www.nlnetlabs.nl/projects/nsd/about/)
10. [Unbound-official documents](https://nlnetlabs.nl/documentation/unbound/unbound.conf/)
11. [Unbound DNS server in chroot](http://sysadminproject.blogspot.com/2014/03/unbound-dns-server-in-chroot.html)
12. [How to Install and Configure ‘Cache Only DNS Server’ with ‘Unbound’](https://www.tecmint.com/setup-dns-cache-server-in-centos-7/)
13. [Configuring Unbound as a simple forwarding DNS server](https://www.redhat.com/sysadmin/forwarding-dns-2)
14. [Enable logging of DNS queries ](https://snippets.khromov.se/enable-logging-of-dns-queries-in-unbound-dns-resolver/)
