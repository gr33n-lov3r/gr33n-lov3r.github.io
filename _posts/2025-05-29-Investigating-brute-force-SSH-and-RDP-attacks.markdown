---
layout: page
title: Investigating brute force SSH and RDP attacks
permalink: /investigating-brute-force-ssh-and-rdp-attacks
---



This lab is the very first one I completed while following along in a challenge ([MyDFIR's 30 day SOC Analyst challenge](https://youtube.com/playlist?list=PLG6KGSNK4PuBb0OjyDIdACZnb8AoNBeq6&si=j-P3c9dueGik3gG-)) to begin experimenting with a bare bones SOC environment that includes the the victim and attack machines, log forwarding and analysis using Fleet and the ELK stack, and Mythic to establish a C2 server, and osTicket for monitoring alerts. This was done entirely in the cloud using Digital Ocean's resources. The lab is fairly easy to set up and understand, and the majority of attacks today target cloud environments, so it's a good opportunity for any beginner like me to security to quickly cultivate practical experience.

In this lab, I will be investigating brute force SSH and RDP attempts to the victim machines I've set up.

## Setting up the lab


I [signed up for a free $200 in credits with Digital Ocean](https://try.digitalocean.com/freetrialoffer/) (link to DigitalOcean's offer on their site) to create my environment in the cloud. 

My setup was as follows:
- Windows 2022 machine with RDP exposed
- Fleet server for log forwarding
- Ubuntu 22.04 machine with Elastic and Kibana on for querying and analysis
- Mythic C2 server to emulate attack
- osTicket server for alerting and ticketing

<img src="{{ site.baseurl }}/assets/diagram2.png" width="500">

This section won't be very long, as I won't go into detail about the lab setup. It'll be tedious explaining all the individual steps and isn't as important as the investigation. Instead, I'll focus on the highlights and what I learned.




### 1. Creation of VPC and Ubuntu machine
I made my Virtual Private Cloud labeled "MYDFIR-SOC-Challenge" with an IP range of `172.30.0.1-254` before creating the Ubuntu machine. ("MYDFIR-ELK") Both need to be in the same region/datacenter so they can see each other. I also generated an SSH keypair to use to connect to my machine.

<img src="{{ site.baseurl }}/assets/Untitled design(1).png" width="500">


<img src="{{ site.baseurl }}/assets/ssh-auth.png" width="500">


### 2. Installation of Elastic and Kibana
I then began to install Elastic and Kibana on the machine using the following commands:
~~~
# Fetching installation packages
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-9.0.1-amd64.deb https://artifacts.elastic.co/downloads/kibana/kibana-9.0.1-amd64.deb
~~~
<img src="{{ site.baseurl }}/assets/install-elastic-package.png" width="500">

~~~
# Installing both instances
dpkg -i https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-9.0.1-amd64.deb https://artifacts.elastic.co/downloads/kibana/kibana-9.0.1-amd64.deb
~~~

I wrote down the superuser password that is generated for Elastic under "Security Autoconfiguration Information", as it is needed to login later. 
Then, in both Elastic and Kibana's configuration files, I made a couple changes so the instance was accessible to the SOC Analyst laptop. I changed `#network.host` to the public IP address of the Ubuntu machine, and uncommented `#http.port: 9200` so Elastic could communicate with Kibana. 
I also created and modified a firewall in Digital Ocean via "Networking" to allow inbound TCP traffic from my public IP, as well as allowed port `5601` on the machine itself to ensure the web UI was accessible. 

<img src="{{ site.baseurl }}/assets/firewall-allow.png" width="500">

One more thing that is needed before accessing the console is an enrollment token for Kibana, which I got by navigating to `usr/share/elasticsearch/bin` and executing:
~~~
elasticsearch-create-enrollment-token --scope kibana
~~~ 

I entered the enrollment token, the superuser password generated, and after another return to the terminal, the 6-digit verification code requested which can be generated as follows:
~~~
Navigate to usr/share/kibana/bin
Execute kibana-verification-code
~~~
<img src="{{ site.baseurl }}/assets/Untitled design.png" width="500">

### 3. Installation of Windows 2022 Server



 


