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

![image](https://github.com/LauraNaddei/NS-Project/assets/73280653/44e37803-ea51-4100-bec4-8b40a0c04c55)

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

![image](https://github.com/LauraNaddei/NS-Project/assets/73280653/804379cc-2735-4372-be14-7075fa63e3cf)


### Footprinting

In this case scenario we're already connected to the local network, so we've already gathered enough information about our target.

We start by finding out our IP address on the networks using ifconfig command:

![MicrosoftTeams-image (3)](https://github.com/LauraNaddei/NS-Project/assets/73280653/8f70051b-19ec-440f-8df7-e257bc8f0861)

Then, we discover some IPs using then nmap tool with these options:

- “-sn” means “no port scan”.
- “-PE” sends ICMP Echo Request.
- “--send-ip” to not send ARP packets.
  
![MicrosoftTeams-image (4)](https://github.com/LauraNaddei/NS-Project/assets/73280653/2736d144-7fd6-47dd-bb8b-ef007c32e6ea)

### Scanning
We're interested in the open services on the network so we can scan it more aggressively.

If we use simply a TPC SYN scan from nmap, we find vague information. In particular, nmap is used with the following flag:
- -sS: TCP SYN

Scanning for the Protected Web Server:

![MicrosoftTeams-image (5)](https://github.com/LauraNaddei/NS-Project/assets/73280653/66eef6c9-c12d-4cdf-bbd0-9d8dae2f583c)

As we can see all the scanned port are filtered.
When we try to access to 193.20.1.2:

![MicrosoftTeams-image (20)](https://github.com/LauraNaddei/NS-Project/assets/73280653/19bbf498-b37b-4867-9b7c-7523f99b3c8f)

This happens because iptables rules were previously set for the 193.20.1.2 IP (Protected Web Server) in order to provide the access to the Web Server only by the 193.20.1.4 IP (Massimiliano PC) as shown:

![image](https://github.com/LauraNaddei/NS-Project/assets/73280653/24366c5e-b0c6-4abb-b940-4ce1edd11fe1)

Scanning for Massimiliano's host:

![MicrosoftTeams-image (7)](https://github.com/LauraNaddei/NS-Project/assets/73280653/d84aca8b-2cb6-4df4-a77f-76f91bd04211)

Scanning for the SMTP Server:

![MicrosoftTeams-image (6)](https://github.com/LauraNaddei/NS-Project/assets/73280653/dc29d8d2-af94-4e40-9928-7dd2434ef71f)


### Enumeration
Instead, if we explore deeply with a Version Detection scan through nmap, we can obtain service fingerprints on the hosts.
In particular, nmap is used with the following flag:
- -sV: probe open ports to determine service/version info.

Enumeration for Massimiliano's host:

![MicrosoftTeams-image (8)](https://github.com/LauraNaddei/NS-Project/assets/73280653/8d34edc2-1429-4336-b8a9-7b4c00b0e1fa)
On Massimiliano PC, we find an OpenSSH open port.

It was previously opened on Massimiliano PC through the command:

```
\urs\sbin\sshd
```

Enumeration for the SMTP Server:

![MicrosoftTeams-image (9)](https://github.com/LauraNaddei/NS-Project/assets/73280653/4fb81806-0d4d-446f-8710-b85764e92878)

We have now discovered about the SMTP server opened on the 25 port.

Our purpose is to find out Massimiliano's username in the intranet context.

Massimiano's credentials into the intranet context are previously set as:

```

```

The first step is to connect manually to the SMTP server and try verifying some users.

The netcat command is used with the following flags:
- -n, --noodns: do not resolve hostnames via DNS
- -v, --verbose: set verbosity level (can be used several times)
  
![image](https://github.com/LauraNaddei/NS-Project/assets/73280653/e37fa670-c10a-4963-bfe3-fe1116fdd192)

To automatize the research operation, the script "smtp-user-enum.pl" is used to enumerate SMTP users using the VRFY command. It is lauched with a list of possible Massimiliano Allegri usernames generated by AI "unix_users.txt".

![MicrosoftTeams-image (10)](https://github.com/LauraNaddei/NS-Project/assets/73280653/ecc6f4ae-9ba7-499f-8f0c-479abdba0e56)

![MicrosoftTeams-image (11)](https://github.com/LauraNaddei/NS-Project/assets/73280653/13fe99a4-49ba-4128-9552-641909f21408)

We find out the matching with "massimiliano" as username.

### Phishing

Our purpose is stealing Massimiliano's password. 

At first, we use the setoolkit tool with the scope of cloning the Linkedin login web page.

![MicrosoftTeams-image (12)](https://github.com/LauraNaddei/NS-Project/assets/73280653/62361729-518f-42ad-b73a-90f889355d8f)

As shown, any request to the 193.20.1.6 IP is mapped to the fake Linkedin login page and every access to that page will be tracked by setoolkit.

Now, we need to trick Massimiliano into accessing our cloned site by sending him a phishing email:

![MicrosoftTeams-image (14)](https://github.com/LauraNaddei/NS-Project/assets/73280653/8c75f516-1a02-418a-8d60-ba166c93a2f3)

![MicrosoftTeams-image (13)](https://github.com/LauraNaddei/NS-Project/assets/73280653/bf98db13-23c5-4ad8-a1f2-a6fbb77d259d)

Once Massimiliano click the fake link provided by the email, he'll be redirected to the fake login page.

Massimiliano inserts his access credentials and will be redirected to the real Linkedin login page.

![MicrosoftTeams-image (15)](https://github.com/LauraNaddei/NS-Project/assets/73280653/41e2e629-eb65-43a5-9bab-9dd8d50ffd76)

On setoolkit we discover Massimiliano's password:

![MicrosoftTeams-image (16)](https://github.com/LauraNaddei/NS-Project/assets/73280653/bb71f785-b4c4-412e-b963-461954ac232b)


### Local Port Forwarding

We can use Massimiliano OpenSSH open port to set up an ssh tunnel with him. 
The local port forwarding tecnique is used to bypass the firewall by forwarding the client machine port 8181 to the server machine port 80 via the Massimiliano's PC.

![MicrosoftTeams-image (17)](https://github.com/LauraNaddei/NS-Project/assets/73280653/d29225f1-6274-49c2-addf-f696d272fa81)

Now, we are able to bypass the firewall accessing to the Protected Web Server from localhost:8181.

![image](https://github.com/LauraNaddei/NS-Project/assets/73280653/8d78c9aa-bec8-447f-ba30-d4686d3881e1)




