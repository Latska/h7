# H7 Oma moduli

I figured it would be best to do something useful for the project so I decided to try out something I might need in the future: 
Installing & Configuring Squid Proxy Cache Server with Salt.

Squid Proxy Server is a server application that stands between the Internet and the webserver providing reverse proxy services and caching options for websites. For this project I wanted to use Squid Proxy to control Internet access for specific sites and also try out some different configurations out of interest. 

Baby steps of first installation:
![Image](https://i.imgur.com/pVyGoR4.png)


* Creator: Lauri Immonen
* State as of now: Alpha
* Link: https://github.com/Latska/h7


I began the project by instaling Squid Proxy manually:

lauri@latska:~$ sudo apt install -y squid
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done

Moving to the /squid/ directory and checking out what's up:

lauri@latska:~$ cd /etc/squid/
lauri@latska:/etc/squid$ ls
conf.d  errorpage.css  squid.conf


The squid.conf configuration file had about ~8600 lines and I decided it would be best to remove the comments out of it, but to be safe lets take a backup of the .conf at first:

lauri@latska:/etc/squid$ sudo cp squid.conf squidbackup.conf

lauri@latska:/etc/squid$ ls
conf.d  errorpage.css  squidbackup.conf  squid.conf

And remove them comments out by using grep:

lauri@latska:/etc/squid$ sudo grep -o '^[^#]*' squid.conf
acl localnet src 0.0.0.1-0.255.255.255	
acl localnet src 10.0.0.0/8		
acl localnet src 100.64.0.0/10		
acl localnet src 169.254.0.0/16 	
acl localnet src 172.16.0.0/12		
acl localnet src 192.168.0.0/16		
acl localnet src fc00::/7       	
acl localnet src fe80::/10      	
acl SSL_ports port 443
acl Safe_ports port 80		
acl Safe_ports port 21		
acl Safe_ports port 443		
acl Safe_ports port 70		
acl Safe_ports port 210		
acl Safe_ports port 1025-65535	
acl Safe_ports port 280		
acl Safe_ports port 488		
acl Safe_ports port 591		
acl Safe_ports port 777		
acl CONNECT method CONNECT
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access allow localhost manager
http_access deny manager
include /etc/squid/conf.d/*
http_access allow localhost
http_access deny all
http_port 3128
coredump_dir /var/spool/squid
refresh_pattern ^ftp:		1440	20%	10080
refresh_pattern ^gopher:	1440	0%	1440
refresh_pattern -i (/cgi-bin/|\?) 0	0%	0
refresh_pattern .		0	20%	4320

Now that the configuration file is all cleaned up, it's time to make the 



