# pihole-docker
A simple docker-compose file for Pi-Hole + Nginx Proxy Manager + Wireguard (with wg-easy) for a home server.

## Containers used
- Pi-Hole https://github.com/pi-hole/docker-pi-hole 
- Nginx Proxy Manager https://github.com/NginxProxyManager/nginx-proxy-manager 
- wg-easy https://github.com/wg-easy/wg-easy

## What is this for?
A reference for fellow hobbyist to prepare a base setup using Docker for a home server prior to setting up other services.
Reasons for using these containers as a base are as follows:
   - Pi-Hole provides basic adblock, with local DNS and CNAME record for setting up proxy hosts in Nginx Proxy Manager.
   - Nginx Proxy Manager allows use of subdomains instead of IP address to access to services.
   - Wireguard allows remote access to the services from outside the network.

## Remote access
For additional security, the home network is only accessible through Wireguard. As such, the services will only be available when the Wireguard connection is active. It is intentional for the home network not being exposed to the public, and is only allowed to users with Wireguard setup on their devices.

## Pre-requisites
- Access to home network router. To change DNS settings.
- Home network with public IP address. Request from the ISP for public IP address if it's unavailable, otherwise this setup will not work.
- Dynamic DNS, e.g. DuckDNS. Necessary for accessing home network through a subdomain, e.g. [YOUR_SUBDOMAIN].duckdns.org instead of changing the public IP address everytime it switches. Refer to the instructions from the respective Dynamic DNS providers on how to setup on your server. This reference shall use DuckDNS as an example.
- Docker

## Instructions
### Initial setup
1. On your server, place the docker-compose file in the desired directory, e.g. in "pihole"
2. Read through the contents and comments thoroughly, then amend docker-compose.yml according to your requirements
3. Run the following command.

   docker compose up -d

### Pi-Hole
1. On the router, change the your primary DNS to [YOUR_PIHOLE_IP], i.e. IP address of Pi-Hole set in the docker-compose file
2. Get Pi-Hole password using the following command
   docker logs pihole | grep random password
3. Login to Pi-Hole administrator page through a web browser through http://[YOUR_PIHOLE_IP]/admin 
4. Once verified Pi-Hole administrator page is accessible, proceed to setup Nginx Proxy Manager

### Nginx Proxy Manager
#### Preparing SSL certificates
1. Go to http://[YOUR_PIHOLE_IP]:81
2. Create administrator account for Nginx Proxy Manager and login
3. Go to "SSL Certificates" page
4. Click "Add SSL Certificate"
5. Choose "Let's Encrypt"
6. In the "Domain Names" field insert the following:
   - [YOUR_SUBDOMAIN].duckdns.org, *.[YOUR_SUBDOMAIN].duckdns.org
7. Check "DNS Challenge"
8. Choose "DuckDNS" in the "DNS Provider" field
9. In the "Credentials File Content" field insert the following:
   - dns_duckdns_token=[YOUR_DUCKDNS_TOKEN]
10. Agree to the T&C for Let's Encrypt then click "Save"
11. The SSL certificates shall be generated after a few minutes. Otherwise, set and tweak the "Propagation Seconds" e.g. 30 seconds or higher

#### Creating Proxy Host for Nginx Proxy Manager
1. Go to "Proxy Hosts" page
2. Click "Add Proxy Hosts" button
3. Insert to the respective fields as follows:
   - Domain Names: [YOUR_SUBDOMAIN].duckdns.org
   - Forward Hostname / IP": nginxproxymanager
   - Forward Port: 81
4. Check "Block Common Exploits"
5. Click "SSL" then insert the following to the SSL Certificate field
   [YOUR_SUBDOMAIN].duckdns.org, *.[YOUR_SUBDOMAIN].duckdns.org
6. Check "Force SSL", "HTTP/2 Support" then Save

### Pihole
1. Get Pi-Hole password using the following command
   docker logs pihole | grep random password
2. Login to Pi-Hole administrator page through a web browser through http://[YOUR_PIHOLE_IP]/admin
3. In the side pane, click "Settings" > "Local DNS Records"
4. In the "Local DNS Records" section, insert to the respective fields as follows:
   - Domain: [YOUR_SUBDOMAIN].duckdns.org
   - IP Address: [YOUR_PIHOLE_IP]
5. Click "+"
6. Switch off your browser's DNS over HTTPS settings
7. Go to [YOUR_SUBDOMAIN].duckdns.org to verify if the Nginx Proxy Manager page is accessible
8. Once accessible, login to Nginx Proxy Manager proceed to set up the proxy for wg-easy

### Wireguard
1. Login to Nginx Proxy Manager with the administrator account
2. Go to "Proxy Hosts" page
4. Click "Add Proxy Hosts" button
5. Insert to the respective fields as follows:
   - Domain Names: [PREFERRED SUBDOMAIN FOR WIREGUARD].[YOUR_SUBDOMAIN].duckdns.org, e.g. wg.[YOUR_SUBDOMAIN].duckdns.org
   - Forward Hostname / IP": wg-easy
   - Forward Port: 51821
6. Check "Block Common Exploits"
7. Click "SSL" then insert the following to the SSL Certificate field
   - [YOUR_SUBDOMAIN].duckdns.org, *.[YOUR_SUBDOMAIN].duckdns.org
8. Check "Force SSL", "HTTP/2 Support" then Save
9. Login to the Pi-Hole administrator page
10. In the side pane, click "Settings" > "Local DNS Records"
11. In the "Local CNAME Records" section, insert to the respective fields as follows:
   - Domain: [YOUR_SUBDOMAIN].duckdns.org
   - Target: wg.[YOUR_SUBDOMAIN].duckdns.org
12. Click "+"
13. Go to wg.[YOUR_SUBDOMAIN].duckdns.org to verify if the wg-easy login page is accessible
14. Once accessible, login to the wg-easy administrator page using the bcrypt password generated earlier when preparing docker-compose file
15. Click "New" to add the required client, and click the QR code button to get the configuration in QR code for setup on your PC or phone app.
