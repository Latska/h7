# H7 Oma moduli 

## b) Kaikki tehtävät tähän. Listaa suora linkki kuhunkin palautukseesi h1, h2...

[H1](https://latezkii.wordpress.com/2022/04/05/h1-hei-maailma-monet-tilat/)

[H2](https://github.com/Latska/h2/blob/main/h2-package-file-service.md)

[H3](https://github.com/Latska/h3/blob/main/h3versionhallinta.md)

[H4](https://latezkii.wordpress.com/h4-aikajana/)

[H5](https://latezkii.wordpress.com/2022/05/03/h5-uusi-komento/)

[H6](https://latskacom.wordpress.com/h6-akkuna/)

## c) Oma moduli (iso tehtävä). Ratkaise jokin oikean elämän tai keksitty tarve omilla tiloilla/moduleilla. Voit käyttää Salttia tai muuta valitsemaasi modernia keskitetyn hallinnan ohjelmaa. Esitä tulos viimeisellä opetuskerralla, 5-10 min. Live demo olisi kiva. Raportoi modulisi tarkoitus, koodi ja testit.


    * Creator: Lauri Immonen
    * State as of now: Completed
    * Testing environment: Debian 11
    * Link to repo: https://github.com/Latska/squid)
    * Link to course: https://terokarvinen.com/2021/configuration-management-systems-2022-spring/

I already had installed Salt Master & Slave on my system, but [here's a great guide for that](https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/)


I figured it would be best to do something useful for the project so I decided to try out something I might need in the future: 
Installing & Configuring Squid Proxy Cache Server with Salt.


Squid Proxy Server is a server application that stands between the Internet and the webserver providing reverse proxy services and caching options for websites. For this project I wanted to use Squid Proxy to control Internet access for specific sites and also try out some different configurations out of interest. 



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


Now that the configuration file is all cleaned up, it's time to create the /salt/squid/ destination and copy the cleaned up .conf file there:

    lauri@latska:/srv/salt/squid$ cd /etc/squid/
    lauri@latska:/etc/squid$ cd
    lauri@latska:~$ cd /etc/squid/
    lauri@latska:/etc/squid$ sudo mkdir /srv/salt/squid
    lauri@latska:/etc/squid$ sudo cp squid.conf /srv/salt/squid/
    lauri@latska:/etc/squid$ cd /srv/salt/squid
    lauri@latska:/srv/salt/squid$ ls
    squid.conf


First I made the init.sls -file and made it path from the /salt/squid directory to /etc/squid/. I also modified the squid.conf file to change its default port from 3128 to 8080:

    lauri@latska:/srv/salt/squid$ sudoedit init.sls
    lauri@latska:/srv/salt/squid$ cat init.sls

    squid:
      pkg.installed
      
    /etc/squid/squid.conf:
      file.managed:
        - source: salt://squid/squid.conf
 
 
And to test it out by running salt state.apply command:

    lauri@latska:/srv/salt/squid$ sudo salt '*' state.apply squid

    orjatar:

          ID: squid
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed
     Started: 20:37:39.529074
    Duration: 197.881 ms
     Changes:   
----------
          ID: /etc/squid/squid.conf
    Function: file.managed
      Result: True
     Comment: File /etc/squid/squid.conf updated
     Started: 20:37:39.729308
    Duration: 33.784 ms
     Changes:   
     Summary for orjatar
    ------------
    Succeeded: 2 (changed=1)
    Failed:    0
    ------------
    Total states run:     2
    Total run time: 231.665 ms


And also enable port 8080 on ufw:


    lauri@latska:/srv/salt/squid$ sudo ufw allow 8080/tcp
    Rules updated
    Rules updated (v6)

    lauri@latska:/srv/salt/squid$ sudo netstat -plnd
    tcp6       0      0 :::8080                 :::*                    LISTEN      44779/(squid-1)     


There are a lot of different tricks you can do with the Squid Proxy. First I wanted to try if it actually worked by restrict the access to certain websites.
I made a new directory /sites/ and added a restricted_sites file with "www.google.com" in there.

    lauri@latska:/srv/salt/squid$ sudo mkdir sites
    lauri@latska:/srv/salt/squid$ cd sites/
    lauri@latska:/srv/salt/squid/sites$ sudoedit restricted_sites
    lauri@latska:/srv/salt/squid/sites$ cat restricted_sites 
    www.google.com
    https://www.google.com


Add the following to the the squid.conf file: 
The first line is path to the .txt files with allowed IP-addresses.

The second line is the path for the URLS that will be restricted from our IP-addresses

The third line is is the actual rule for the config file.

    acl allowed_ips src "/etc/squid/allowed_ips.txt"  
    acl restrictedsites dstdomain "/etc/squid/restricted_sites"
    http_access deny restricted_sites 


And also added the following to the init.sls -file:

    /etc/squid/restricted_sites:
      file.managed:
        - source: salt://squid/sites/restricted_sites

    /etc/squid/allowed_ips.txt:
      file.managed:
        - source: salt://squid/allowed_ips.txt

 
    
    
And to try out the changes by salt state.apply command:

      lauri@latska:/srv/salt/squid$ sudo salt '*' state.apply squid
      orjatar:
      changes:  
      Summary for orjatar
      ------------
      Succeeded: 4 (changed=3)
      Failed:    0
      ------------
      Total states run:     4
      Total run time: 219.061 ms

The Salt states seemed to be working, but now for testing the actual proxy settings if it's working by using the 'curl' command and also with Fire fox:

    lauri@latska:/srv/salt/squid$ curl www.google.com

     /*
     * Copyright (C) 1996-2020 The Squid Software Foundation and contributors
     *
     * Squid software is distributed under GPLv2+ license and includes
     * contributions from numerous individuals and organizations.
     * Please see the COPYING and CONTRIBUTORS files for details.
     */

![image](https://user-images.githubusercontent.com/103587811/168886483-6b61a884-5393-4f7c-8fdd-7146aa64f788.png)


And to test out that other sites are available:

![image](https://user-images.githubusercontent.com/103587811/168886733-19c2924f-d0bc-4175-9dfc-2d11da15ee02.png)


Looks like it's working as planned. Next up I wanted to try blocking sites by simply using words such as 'google', 'yle' or 'fi' instead of actual URLs:
First I made new file 'blocked_sites' in /squid/sites with those blockable words.

    lauri@latska:/srv/salt/squid$ cd sites/
    lauri@latska:/srv/salt/squid/sites$ sudoedit blocked_sites
    [sudo] password for lauri: 
    lauri@latska:/srv/salt/squid/sites$ cat blocked_sites 
    google
    yle
    fi


Then added the following lines to the squid.conf -file:

    acl blocked_sites url_regex -i "/etc/squid/blocked_sites" 
    http_access deny blocked_sites 


Added the following to the init.sls file:

  /etc/squid/blocked_sites:
    file.managed:
      - source: salt://squid/sites/blocked_sites

Applied the changes with state.apply -command and restarted Squid:


    lauri@latska:/srv/salt/squid$ sudo salt '*' state.apply squid
    Summary for orjatar
    ------------
    Succeeded: 5 (changed=2)
    Failed:    0
    ------------
    lauri@latska:/srv/salt/squid$ sudo systemctl restart squid


And to test out if the new proxy settings are working: Looks like https://yle.fi is no longer working:

![image](https://user-images.githubusercontent.com/103587811/168889785-cf7ca694-7e75-47b9-b52d-2df2860a73fe.png)


As well as iltalehti.fi (as we set 'fi' in the banned_sites list), but wikipedia.org is working as it should:
![image](https://user-images.githubusercontent.com/103587811/168890421-50f4cd22-a4fa-4f41-884e-ce7444614a2a.png)


The last test was only allow access to a certain sites from our network:

Made a new file with the allowed sites (and also removed the 'fi' from banned words list:

    lauri@latska:/srv/salt/squid$ cd sites/
    lauri@latska:/srv/salt/squid/sites$ sudo micro allowed_sites
    lauri@latska:/srv/salt/squid/sites$ cat allowed_sites 
    .terokarvinen.com
    .haaga-helia.fi

    lauri@latska:/srv/salt/squid/sites$ sudo micro blocked_sites 
    lauri@latska:/srv/salt/squid/sites$ cat blocked_sites 
    google
    yle


Added the following to the squid.conf file, meaning that everything else should be blocked but those sites we just determined:

    acl allowed_sites dstdomain "/etc/squid/allowed_sites"
    http_access allow allowed_ips allowed_sites


And the following to the init.sls -file:

    /etc/squid/allowed_sites:
      file.managed:
        - source: salt://squid/sites/allowed_sites


And try out the changes with salt state.apply command and restart the Squid.service:

      lauri@latska:/srv/salt/squid$ sudo salt '*' state.apply squid
      orjatar:
      Summary for orjatar
      ------------
      Succeeded: 6 (changed=3)
      Failed:    0
      ------------
      Total states run:     6

    lauri@latska:/srv/salt/squid$ sudo systemctl restart squid


And to test out the newest proxy settings. Looks like we can access haaga-helia.fi & terokarvinen.com, which were the only ones we allowed:

![image](https://user-images.githubusercontent.com/103587811/168892159-0ccd2df0-8535-45ae-a1b0-369971875752.png)



And to test out any other website, unable to connect:

![image](https://user-images.githubusercontent.com/103587811/168905923-e79a42b0-4142-4648-a93d-679ba34a8e98.png)




Now for the very last task I decided to add state that would reload the squid.service if changes are made in the squid.conf file:

    squid:    
      service.running:
        - enable: True
        - reload: True
        - watch:
          - file: /etc/squid/squid.conf
          
          
And tested it by editing the .conf file for once more and running salt state.apply command:


    ----------
              ID: squid
        Function: service.running
          Result: True
         Comment: Service reloaded
         Started: 00:47:28.031931
        Duration: 68.358 ms
         Changes:   
                  ----------
                  squid:
                      True

    Summary for orjatar
    ------------
    Succeeded: 7 (changed=2)
    Failed:    0
    ------------
    Total states run:     7






# The final productions:

  ## squid.conf:

      squidie:
        pkg.installed:
          - pkgs:
            - squid

      /etc/squid/squid.conf:
        file.managed:
          - source: salt://squid/squid.conf

      /etc/squid/restricted_sites:
        file.managed:
          - source: salt://squid/sites/restricted_sites

      /etc/squid/blocked_sites:
        file.managed:
          - source: salt://squid/sites/blocked_sites

      /etc/squid/allowed_sites:
        file.managed:
          - source: salt://squid/sites/allowed_sites

      /etc/squid/allowed_ips.txt:
        file.managed:
          - source: salt://squid/allowed_ips.txt

      squid:    
        service.running:
          - enable: True
          - reload: True
          - watch:
            - file: /etc/squid/squid.conf



## Salt:

      lauri@latska:/srv/salt/squid$ tree
      .
      ├── allowed_ips.txt
      ├── init.sls
      ├── sites
      │   ├── allowed_sites
      │   ├── blocked_sites
      │   └── restricted_sites
      └── squid.conf

      1 directory, 6 files

## init.sls:

![image](https://user-images.githubusercontent.com/103587811/168916355-5ee37b6e-cab6-4f80-ab18-e4fd5bbeef06.png)


## Sources:

- Tero Karvinen - Configuration management systems 2022 
https://terokarvinen.com/2021/configuration-management-systems-2022-spring/

- STATE MODULES - https://docs.saltproject.io/en/latest/ref/states/all/

- squid-cache.org - http://www.squid-cache.org/

- How To Setup and Configure a Proxy Server – Squid Proxy - https://devopscube.com/setup-and-configure-proxy-server/

- How to Set Up Squid Proxy for Private Connections on Ubuntu 20.04 - https://www.digitalocean.com/community/tutorials/how-to-set-up-squid-proxy-on-ubuntu-20-04
























