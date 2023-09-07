# Network Security-Project

## Background Scenario

Our laboratory aims to simulate a real-world scenario where Luciano, an attacker, attempts to access a protected server containing sensitive information. For security purposes, this server is configured to allow access solely from a specific IP address associated with the system's admin, Massimiliano.

During the demonstration scenario, we discovered these IPs:

- 193.20.1.1 - Company Network

- 193.20.1.2 - Sensitive Information Web Server

- 193.20.1.3 - SMTP server

- 193.20.1.4 - Admin Employee (Massimiliano) PC on company network

- 193.20.1.5 - Luciano PC on company network


## Instruction: what to do
1. Footprinting: gather information online about the organization and its systems.
2. Scanning: scan vulnerable point of access.
3. Enumeration: intrusive probing of vulnerable services.
4. Exploitation: attack those potential vulnerability.

## Footprinting

In this case scenario we're already connected to the local network, so we've already gathered enough information about our target.

We start by finding out our IP address on the networks using ifconfig command:
![MicrosoftTeams-image (3)](https://github.com/LauraNaddei/NS-Project/assets/73280653/8f70051b-19ec-440f-8df7-e257bc8f0861)

Then, we discover some IPs using then nmap tool with these options:

- “-sn” means “no port scan”.
- “-PE” sends ICMP Echo Request.
- “--send-ip” to not send ARP packets.
  
![MicrosoftTeams-image (4)](https://github.com/LauraNaddei/NS-Project/assets/73280653/2736d144-7fd6-47dd-bb8b-ef007c32e6ea)

## Scanning
We're interested in the Sensitive Information Web Server, in the SMTP Server and in Massimiliano PC, so we can scan more aggressively them to find information about any open ports.

If we use simply a TPC SYN scan from nmap, we find vague information.

Scanning for the Sensitive Information Web Server:

![MicrosoftTeams-image (5)](https://github.com/LauraNaddei/NS-Project/assets/73280653/66eef6c9-c12d-4cdf-bbd0-9d8dae2f583c)

Scanning for the SMTP Server:
![MicrosoftTeams-image (6)](https://github.com/LauraNaddei/NS-Project/assets/73280653/dc29d8d2-af94-4e40-9928-7dd2434ef71f)

Scanning for Massimiliano's host:
![MicrosoftTeams-image (7)](https://github.com/LauraNaddei/NS-Project/assets/73280653/d84aca8b-2cb6-4df4-a77f-76f91bd04211)

