Week 9 Project: Honeypot
Due: Sunday, April 2nd at 11:59pm

Summary: Setup a honeypot and provide a working demonstration of its features.

Background:

A honeypot is a decoy application, server, or other networked resource that intentionally exposes insecure features which, when exploited by an attacker, will reveal information about the methods, tools, and possibly even the identity of that attacker. Honeypots are commonly used by security researchers to understand the threat landscape facing developers and system administrators, collecting data that might include:

Information about sources of malicious network traffic such as IP addresses, geographic origin, targeted ports, etc.
Information used to harden resources against email spammers
Malware samples
DB vulnerabilities such as SQLI techniques
There are two broad types of honeypots:

Low-interaction honeypots provide simulations of target resources, typically using emulation or virtualization, to reduce resource consumption, simplify configuration and deployment, and provide solid containment features
High-interaction honeypots expose non-simulated target resources in a way that more closely imitates a production environment to attract more sophisticated attackers and understand more complicated exploitation routes
For example, a low-interaction honeypot might emulate a server capable of accepting SSH connections through a combination of exposed ports and decoy responses, whereas the high-interaction version would feature an actual SSH server possibly misconfigured in some way that makes it vulnerable. In the low-interaction example, attempted exploitation would quickly lead to a dead end for the attacker, perhaps revealing only an IP address and a few attempted commands to the honeypot's maintainer. In the high-interaction example, the attacker would potentially be able to compromise the server, wasting more time and giving away more information about his or her goals.

Overview:

In this assignment, you will stand up a basic honeypot and demonstrate its effectiveness at detecting and/or collecting data about an attack. You are free to take any approach you wish that demonstrates the following basic principles:

Successful configuration and deployment of a network-accessible honeypot server with two primary features:
An attack surface that is vulnerable or exposed in some way to network-based attacks
A network security feature such as an IDS configured to detect and log such attacks
Illustration of at least one attack against the honeypot that can be detected or logged in a way that captures information about the attack or the attacker
There are a number of different ways you might approach this project. Here are a few examples to consider, from least to most effort:

A pre-packaged open-source honeypot than can detect and capture port scans conducted via nmap
A WordPress instance running one or more outdated, vulnerable plugins, and an IDS capable of detecting footprinting attempts via wpscan
A custom-configured server that features a version of the Globitek application that contains several SQLI vulnerabilities and an IDS that is able to detect probing via sqlmap
Any one of these ideas would meet the basic requirements, but all of them could be extended in a way that makes them more sophisticated. For example, the WordPress instance could be running a custom version of a vulnerable plugin you've researched that's been modified in a way that preserves the basic vulnerability but limits the potential damage. For example, could you think of a way to modify the Reflex Gallery 3.1.3 we exploited in week 8 that allows the attacker to deliver a meterpreter payload but prevent it from being executed or deleted?

Requirements:

As said above, there are a few basic requirements for this project, but you shouldn't stop at the minimum. You should direct most of your time and effort toward learning about some of the topics we've worked on, such as:

Installing and configuring an IDS or firewall
Logging and analyzing network traffic
Exposing specific vulnerabilities
Here's a list that includes but isn't limited to possible features:

Required
Honeypot: a network-accessible server setup including both a target and a mechanism for capturing data on exploitation attempts
A basic writeup (250-500 words) on the README.md desribing the overall approach, resources/tools used, findings
A specific, reproducible honeypot setup, ideally automated. There are several possibilities for this:
A Vagrantfile or Dockerfile which provisions the honeypot as a VM or container
A bash script that installs and configures the honeypot for a specific OS
Alternatively, detailed notes added to the README.md regarding the setup, requirements, features, etc.
Demonstration: at least one proof-of-concept of an attack against the honeypot that was detected / intercepted
A basic writeup of the attack (what offensive tools were used, what specifically was detected by the honeypot)
An example of the data captured by the honeypot (example: IDS logs including IP, request paths, alerts triggered)
A screen-cap of the attack being conducted
Optional
Honeypot
HTTPS enabled (self-signed SSL cert)
A web application with both authenticated and unauthenticated footprint
Database back-end
Custom exploits (example: intentionally added SQLI vulnerabilities)
Custom traps (example: modified version of known vulnerability to prevent full exploitation)
Custom IDS alert (example: email sent when footprinting detected)
Custom incident response (example: IDS alert triggers added firewall rule to block an IP)
Demonstration
Additional attack demos/writeups
Captured malicious payload
Enhanced logging of exploit post-exploit activity (example: attacker-initiated commands captured and logged)
Resources:

There are many free and open-source honeypots and resources for building and deploying them available online. You are free to use any and all of these tools to fulfill the basic requirements. Here are some links to get you started:

Honeypots
Awesome Honeypots - A curated list of awesome honeypots, tools, components and much more
Modern Honey Network - The Modern Honey Network project makes deploying and managing secure honeypots extremely simple
HoneyPress - A WordPress honeypot in a docker container
IDS / Firewall Tools
Suricata IDS
Snort IDS
pfSense Firewall - Most popular UI for pf
Submitting Assignments: Submitted through Github, check out the Submitting Assignments page for more details.

Be sure to include the README containing a writeup about your honeypot setup and demonstration of an attack, including a GIF walkthrough
Include any scripts used to provision or configure the honeypot or conduct the attack(s)
Include any log files generated by the honeypot that demonstrate detected attack(s)
Any code for a custom target or modified version of a known target
Use this README template in order to have a complete README.
Getting Started
There a lots of resources online available in setting up a honeypot, but it can be a bit confusing to know where to start. The Modern Honey Network (MHN) provides a streamlined approach to setting up a low-interaction honeypot. To get you started, the following instructions walk through a local setup of an MHN server/honeypot using Vagrant. Instructions are adapted from the MHN wiki. You need to have Vagrant installed with a VM provider such as VirtualBox.

On Your Host Machine
Start by cloning the MHN source and launching via Vagrant:

$ git clone https://github.com/threatstream/mhn.git
$ cd mhn
$ vagrant up
This should create two VMs:

honeypot - the honeypot VM, aka the target
server - the VM running the admin console for monitoring data collected by the honeypot
Verify the two VMs are up and running:

$ vagrant global-status
id       name     provider   state    directory
--------------------------------------------------------------------------------------------------------------------------
924087e  server   virtualbox running  /Users/user/week9/mhn
e91f356  honeypot virtualbox running  /Users/user/week9/mhn
Confirm the IP addresses of each. The values below are statically assigned in the Vagrant file, so your output should match the below:

$ vagrant ssh server -c "hostname -I | cut -d' ' -f2" 2>/dev/null
# 10.254.254.100
$ vagrant ssh honeypot -c "hostname -I | cut -d' ' -f2" 2>/dev/null
# 10.254.254.101
Open shell on server:

$ vagrant ssh server
Inside the server VM
After executing the vagrant ssh server command above, the command prompt will change to vagrant@mhn-server:~, indicating you're now inside the server VM. Start by opening a superuser shell and installing git:

$ sudo su -
$ apt-get install -y git
At this point we need to clone the mhn repo again inside the server vm. Due to an outstanding issue we will need to use a patched fork instead of the primary repo:

$ cd /opt/
$ git clone https://github.com/0x7fff9/mhn.git
# Cloning into 'mhn'...
# ...
$ cd mhn
# pwd should now be /opt/mhn
$ git checkout 0x7fff9-encoding_patch
Branch 0x7fff9-encoding_patch set up to track remote branch 0x7fff9-encoding_patch from origin.
Switched to a new branch '0x7fff9-encoding_patch'
Now, from /opt/mhn, launch the pre-requisite install scripts:

$ cd scripts
# pwd should now be /opt/mhn/scripts
$ ./install_hpfeeds.sh ; ./install_mnemosyne.sh ; ./install_honeymap.sh
This will take ~10 mins. After this, check the supervisor process to make it all components were installed

$ supervisorctl status
geoloc                           RUNNING    pid 29334, uptime 0:02:22
honeymap                         RUNNING    pid 29335, uptime 0:02:22
hpfeeds-broker                   RUNNING    pid 10253, uptime 0:07:33
mnemosyne                        RUNNING    pid 28222, uptime 0:06:09
Important: Since this is a private network deployment, you need to change a mnemosyne config option to support this. In the server vm, edit /opt/mnemosyne/mnemosyne.cfg and change ignore_rfc1918 to False:

[normalizer]
ignore_rfc1918 = False
Now restart mnemosyne and run the final server install script:

$ supervisorctl restart mnemosyne
# pwd should still be /opt/mhn/scripts
$ ./install_mhnserver.sh
The script will prompt you for values (choose whatever you like for Superuser password, otherwise blank indicates default):

Do you wish to run in Debug mode?: y/n n
Superuser email: YOUR-EMAIL@YOUR-SITE.com
Superuser password: 
Superuser password: (again): 
Server base url ["http://1.2.3.5"]: http://10.254.254.100
Honeymap url ["http://1.2.3.5:3000"]: http://10.254.254.100:3000
Mail server address ["localhost"]: 
Mail server port [25]: 
Use TLS for email?: y/n y
Use SSL for email?: y/n y
Mail server username [""]:  
Mail server password [""]: 
Mail default sender [""]: 
Path for log file ["/var/log/mhn/mhn.log"]: 
The script will complete in about ~10 mins. When it completes, exit the superuser shell and the server VM by running the exit command twice to return to your host machine:

root@mhn-server:/opt/mhn/scripts# exit
# logout
vagrant@mhn-server:~$ exit
# logout
# Connection to 127.0.0.1 closed.
On Your Host Machine
Access the MHN admin console via a browser at http://10.254.254.100 and login with the email / password you specified in the previous step.

Click around to familiarize yourself with the interface.

Go to the Deploy page and in the Select Script input, select Ubuntu - Dionaea and copy the Deploy Command snippet to your clipboard. Dionaea is a low-interaction honeypot designed to trap malware (though we'll just look at using it to detect port scanning).

Back in the terminal, from your host machine, open a shell into the honeypot VM:

$ vagrant ssh honeypot
The prompt should change again.

Inside the honeypot VM
Open a superuser shell and execute the deploy command you copied from the browser page:

$ sudo su -
# command copied from "Deploy Command"
$ wget "http://10.254.254.100/api/script/?text=true&script_id=2" -O deploy.sh && sudo bash deploy.sh http://10.254.254.100 DuGSk2KT
The script will run for a few minutes and will eventually prompt you twice with File to Patch: to which you should supply the following values, respectively:

/etc/dionaea/dionaea.conf
/usr/lib/dionaea/python/dionaea/ihandlers.py
The text leading up to this was:
--------------------------
|--- /etc/dionaea/dionaea.conf
|+++ /etc/dionaea/dionaea.conf.new
--------------------------
File to patch: /etc/dionaea/dionaea.conf

...

The text leading up to this was:
--------------------------
|
|--- /usr/lib/dionaea/python/dionaea/ihandlers.py
|+++ /usr/lib/dionaea/python/dionaea/ihandlers.py.new
--------------------------
File to patch: /usr/lib/dionaea/python/dionaea/ihandlers.py
When it completes, exit the superuser shell and the honeypot VM by running the exit command twice to return to your host machine:

root@mhn-honeypot:~# exit
logout
vagrant@mhn-honeypot:~$ exit
logout
Connection to 127.0.0.1 closed.
On Your Host Machine
Now, in the browser, go to the Sensors page and you should see mhn-honeypot-dionaea listed.

Check out the Attacks page next; you should see no data...yet.

Attack!
Let's try a simple attack against the honeypot VM using nmap to see how it is detected.

You could stage the attack from your host machine or your Kali container or VM, which may already have nmap installed (in which case, skip the setup step).

Setup

For simplicity's sake, we'll attack from the server VM (if you're running Windows, this may be the easiest way to get nmap installed). Starting from the host, open a Vagrant shell into the server VM and install nmap via apt:

$ vagrant ssh server
vagrant@mhn-server:~$ sudo apt-get install -y nmap
Attack with nmap

Run a scan against the honeypot VM using nmap:

vagrant@mhn-server:~$ nmap -sV -P0 10.254.254.101

Starting Nmap 5.21 ( http://nmap.org ) at 2017-04-08 18:55 UTC
Leave nmap running and go back to the Attacks page in the console. You should see evidence of the nmap scan appearing in the console.
