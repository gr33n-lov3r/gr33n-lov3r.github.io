---
layout: page
title: Investigating brute force SSH and RDP attacks
permalink: /investigating-brute-force-ssh-and-rdp-attacks
---
# Investigating brute force SSH and RDP attacks


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

<img src="{{ site.static_files }}\assets\diagram2.png" width="300">

This section won't be very long, as I won't go into detail about the lab setup. It'll be tedious explaining all the individual steps and isn't as important as the investigation. Instead, I'll focus on the highlights and what I learned.

I made my Virtual Private Cloud labeled "MYDFIR-SOC-Challenge" with an IP range of `172.30.0.1-254` before creating the Ubuntu machine. ("MYDFIR-ELK") Both need to be in the same region/datacenter so they can see each other.

<img src="{{ site.static_files }}\docs\assets\Untitled design(1).png" width="300">

So I could SSH into my machine, I generated an SSH keypair using `ssh-keygen` and inputting the key into the requested field.

<img src="{{ baseurl }}\docs\assets\ssh-auth.png" width="300">


I then began to install Elastic and Kibana on the machine using the following commands:
~~~
# Fetching installation packages
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-9.0.1-amd64.deb https://artifacts.elastic.co/downloads/kibana/kibana-9.0.1-amd64.deb
~~~
<img src="{{ site.baseurl }}\docs\assets\install-elastic-package.png" width="300">

~~~
# Installing both instances
dpkg -i https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-9.0.1-amd64.deb https://artifacts.elastic.co/downloads/kibana/kibana-9.0.1-amd64.deb
~~~


Several configurations had to be made. Before starting anything, it's important to write down the Elastic superuser password under "Security Autoconfiguration Information" so one can login to the console.

Elastic and Kibana's configuration files both have to be changed in order for the instance to be accessible on the "SOC Analyst" laptop, as well as the firewall. In the `elasticsearch.yml` and `kibana.yml` configuration files, `#network.host` must be uncommented and changed to the public IP address of the Ubuntu machine, along with `#http.port: 9200`, what Elastic uses for communication with Kibana and agents.

Navigating to Networking in my DO dashboard shows an option to create a firewall for available machines. Using *my* public IP address, I allowed all inbound TCP traffic. And since Kibana is accessed through `port #5601`, I also allowed that on the virtual machine itself so I could use the web UI.

<img src="{{ site.static_files }}\docs\assets\firewall-allow.png" width="300">

I then navigated to `usr/share/elasticsearch/bin` and executed:
`elasticsearch-create-enrollment-token --scope kibana`, generating the enrollment token for registration.


After starting both services and accessing the console, I start entering information saved from earlier. 

When asked for the verification code, I go to `usr/share/kibana/bin` and execute `kibana-verification-code`, which generates a 6-digit code. The password generated after installing Elastic from before is also needed here.

<img src="{{ site.static_files }}\docs\assets\Untitled design.png" width="300">


 


