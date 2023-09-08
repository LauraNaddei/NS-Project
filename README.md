# Over the fire(wall) [Network Security - Project]

Network Security project about Phishing and SSH.

## Background Scenario

Our laboratory aims to simulate a real-world scenario where Luciano, an attacker, attempts to access a protected server containing sensitive information. For security purposes, this server is configured to allow access solely from a specific IP address associated with the system's admin, Massimiliano.

Objectives:

1. Analyze the network to identify Massimiliano's IP.
2. Luciano must find potential vulnerabilities or techniques to bypass the IP-based firewall set by Massimiliano.
3. Gain access to the protected server and retrieve the sensitive information from it.

## Scenario 
![Intranet](https://github.com/LauraNaddei/NS-Project/blob/main/images/intranet.png)


During the demonstration scenario, we use these IPs:

- 193.20.1.1 - Company Network

- 193.20.1.2 - Protected Web Server

- 193.20.1.3 - SMTP server

- 193.20.1.4 - Admin Employee (Massimiliano) PC on company network

- 193.20.1.5 - Luciano PC on company network (Firefox and SSH client)

- 193.20.1.6 - Luciano PC on company network (Hack tools including nmap, netcat and setoolkit)
  
## Phases
1. Footprinting: gather information online about the organization and its systems.
2. Scanning: scan vulnerable point of access.
3. Enumeration: intrusive probing of vulnerable services.
4. Exploitation: attack those potential vulnerability.

## Docker HUB
For our purposes a new docker image is created for the attacker's machine following these steps: 

1. A Dockerfile is created as follows:

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/DockerFile.png)

2. The image "hacker_host" is created through the "docker build" command:
   
```
docker build -t hacker_host .
```

3. The new image is pushed on our public Docker HUB repository for future use:
   
```
docker login
docker tag hacker_host lauranaddei/hacker_host
docker push lauranaddei/hacker_host
```

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/DockerHUB.png)

### Footprinting

In this case scenario we're already connected to the local network, so we've already gathered enough information about our target.

We start by finding out our IP address on the networks using ifconfig command:

![image](https://github.com/LauraNaddei/NS-Project/blob/main/hacker_ifconfig.png)

Then, we discover some IPs using then nmap tool with these options:

- “-sn” means “no port scan”.
- “-PE” sends ICMP Echo Request.
- “--send-ip” to not send ARP packets.

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/footprinting.png)

### Scanning
We're interested in the open services on the network so we can scan it more aggressively.

If we use simply a TPC SYN scan from nmap, we find vague information. In particular, nmap is used with the following flag:
- -sS: TCP SYN

Scanning for the Protected Web Server:

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/scanning_protected_server.png)

As we can see all the scanned port are filtered.
When we try to access to 193.20.1.2:

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/denied_access.png)

This happens because iptables rules were previously set for the 193.20.1.2 IP (Protected Web Server) in order to provide the access to the Web Server only by the 193.20.1.4 IP (Massimiliano PC) as shown:

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/IPtables_setting.png)

Scanning for Massimiliano's host:

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/scanning_Massimiliano.png)

Scanning for the SMTP Server:

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/scanning_SMTP.png)

### Enumeration
Instead, if we explore deeply with a Version Detection scan through nmap, we can obtain service fingerprints on the hosts.
In particular, nmap is used with the following flag:
- -sV: probe open ports to determine service/version info.

Enumeration for Massimiliano's host:

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/enumeration_Massimiliano.png)

On Massimiliano PC, we find an OpenSSH open port.

It was previously opened on Massimiliano PC through the command:

```
\urs\sbin\sshd
```

Enumeration for the SMTP Server:

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/enumeration_SMTP.png)

We have now discovered about the SMTP server opened on the 25 port.

Our purpose is to find out Massimiliano's username in the intranet context.

Massimiano's credentials into the intranet context are previously set as:

```
adduser massimiliano
password: forzajuve
```

The first step is to connect manually to the SMTP server and try verifying some users.

The netcat command is used with the following flags:
- -n, --noodns: do not resolve hostnames via DNS
- -v, --verbose: set verbosity level (can be used several times)

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/netcat(SMTP).png)  

To automatize the research operation, the script "smtp-user-enum.pl" is used to enumerate SMTP users using the VRFY command. It is lauched with a list of possible Massimiliano Allegri usernames generated by AI "unix_users.txt".

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/unix_user.txt.png)

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/Script+VRFY.png)

We find out the matching with "massimiliano" as username.

### Phishing

Our purpose is stealing Massimiliano's password. 

At first, we use the setoolkit tool with the scope of cloning the Linkedin login web page.

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/setoolkit.png)

As shown, any request to the 193.20.1.6 IP is mapped to the fake Linkedin login page and every access to that page will be tracked by setoolkit.

Now, we need to trick Massimiliano into accessing our cloned site by sending him a phishing email:

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/mail_phishing.png)

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/phishing_link.png)

Once Massimiliano click the fake link provided by the email, he'll be redirected to the fake login page.

Massimiliano inserts his access credentials and will be redirected to the real Linkedin login page.

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/Linkedin_fake_page.png)

On setoolkit we discover Massimiliano's password:

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/credentials_stealing.png)

### Local Port Forwarding

We can use Massimiliano OpenSSH open port to set up an ssh tunnel with him. 
The local port forwarding tecnique is used to bypass the firewall by forwarding the client machine port 8181 to the server machine port 80 via the Massimiliano's PC.

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/LocalPortForwarding.png)

Now, we are able to bypass the firewall accessing to the Protected Web Server from localhost:8181.

![image](https://github.com/LauraNaddei/NS-Project/blob/main/images/BypassFirewall.png)





