# Secure Load-Balanced Network Configuration Steps

## 1. Download and Setup 2 VMs on VMWare using Ubuntu Server 20.04
Download and setup 2 VMs using ubuntu server 20.04. Configure NAT, DHCP, Port forwarding for the two VMs in VMWare setting and ensure openssh is downloaded and activated for remote connections on these 2 machines.

Setup ufw to filter traffic

``` bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
```

## 2. DVWA Setup with Docker 
Install Docker and run DVWA on both VM1 and VM2. 

```bash
sudo apt update && sudo apt install docker.io
sudo docker run -d -p 80:80 vulnerables/web-dvwa
```

## 3. Load Balancer Configuration (Middle VM)
Install and configure Nginx on Kali Linux as a reverse proxy.

### Installed Nginx
```bash
sudo apt update
sudo apt install nginx 
```

### Configured `/etc/nginx/nginx.conf`
```nginx
http {
  upstream dvwa_servers {
    server 192.168.58.128; # IP address of server 1
    server 192.168.58.129; # IP address of server 2
  }

  server {
    listen 80;
    location / {
      proxy_pass http://dvwa_servers;
    }
  }
}
```

### Restarted Nginx to apply the configuration
```bash
sudo systemctl restart nginx
```

## 4. Configure Port Forwarding
Configure VMware NAT to forward traffic from the host to VM3.

- Forwarded host port `8080` to VM3 port `80`.
- Accessed DVWA via: `http://localhost:8080`.


### Connected from Host
```powershell
ssh username@192.168.58.128
```

## 5. UFW Firewall Configuration (VM1, VM2)
Set up UFW to secure incoming traffic on VM1 and VM2.

```bash
sudo ufw default deny incoming
sudo ufw allow from 192.168.58.130 to any port 80
sudo ufw enable
```

## 6. Apache Access Log Analysis
Monitor Apache logs inside the DVWA Docker containers.

```bash
sudo docker logs <container_id>
```

- Verified traffic patterns and checked for blocked IPs.