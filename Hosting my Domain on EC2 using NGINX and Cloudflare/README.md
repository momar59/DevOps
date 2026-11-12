# Hosting a Custom Domain on AWS EC2 with Nginx and Cloudflare

## Project Overview

In this project, I deployed my personal website using an AWS EC2 instance, configured Nginx as the web server, connected a custom domain purchased through Cloudflare, and enabled HTTPS using an SSL certificate from Let's Encrypt and Certbot.

This project gave me practical experience with cloud infrastructure, Linux server administration, DNS configuration, web server management and SSL/TLS security.

**Live website:** https://mohamedomar.dev  *(The EC2 instance will only be kept running when required and may be shut down during periods of inactivity to minimise cloud expenditure)*





<img width="1920" height="977" alt="my webpage hosted on ec2" src="https://github.com/user-attachments/assets/800fdab9-14fd-4abe-8020-0559cfb5e5fa" />


## Technologies and Services Used

- **Amazon Web Services (AWS):** EC2 virtual server
- **Ubuntu:** Operating system running on the EC2 instance
- **Nginx:** Web server used to host the website
- **Cloudflare:** Domain management and DNS configuration
- **Let's Encrypt:** Free SSL/TLS certificate provider
- **Certbot:** Automated SSL certificate installation and renewal
- **SSH:** Secure remote access to the EC2 instance
- **HTML/CSS:** Website customisation

## Architecture
<img width="1774" height="886" alt="My Website Architecture Flowchart" src="https://github.com/user-attachments/assets/2322fd99-6b73-4fda-b6e9-dd48de97a653" />



## 1. Creating an AWS EC2 Instance

I started by creating an EC2 instance through the AWS Management Console.

I selected:
- Name as 'Website-server-prod'
- Ubuntu Server as the operating system
- A t3.micro instance
- A key pair to securely access the server through SSH
- A security group to control inbound network traffic

After launching the instance, I noted its public IPv4 address, which I later used to configure my domain's DNS records.

<img width="1567" height="253" alt="ec2 running" src="https://github.com/user-attachments/assets/9ffb45df-ccdd-4fbe-8a6f-371dd021821f" />


## 2. Configuring the Security Group

I configured the EC2 security group to allow the network traffic required to host and access the website.

| Type | Port | Source | Purpose |
|---|---:|---|---|
| SSH | 22 | My IP address | Secure server access |
| HTTP | 80 | 0.0.0.0/0 | Website traffic |
| HTTPS | 443 | 0.0.0.0/0 | Secure website traffic |

I restricted SSH access to my own IP address while allowing public HTTP and HTTPS traffic so visitors could access the website.

<img width="1572" height="301" alt="security group rules" src="https://github.com/user-attachments/assets/be44a001-b97c-40cf-b060-761e5a8cfa6e" />



## 3. Connecting to the EC2 Instance Using SSH

I configured the minimum required permissions for the .pem (private ssh key) file to ensure maximum restriction, granting only myself (owner) the 'read' permission.

```bash
chmod 400 my-key.pem
```

I then connected to the instance from my local terminal using SSH and my downloaded private key.

```bash
ssh -i my-key.pem ubuntu@MY_EC2_PUBLIC_IP
```

Once connected, I could manage the server directly through the terminal.

```bash
ssh -i my-key.pem ubuntu@MY_EC2_PUBLIC_IP
Welcome to Ubuntu 26.04.1 LTS (GNU/Linux 7.0.0-1016-aws x86_64)
```


## 4. Installing and Configuring Nginx web server

After connecting to the server, I updated the Ubuntu package list:

```bash
sudo apt update
```

I installed Nginx:

```bash
sudo apt install nginx -y
```

I checked that the service was running:

```bash
sudo systemctl status nginx
```

I also enabled Nginx to start automatically when the server boots:

```bash
sudo systemctl enable nginx
```

To confirm that the web server was working, I visited the EC2 public IPv4 address in my browser. The default Nginx welcome page confirmed that the server was responding.

<img width="1566" height="270" alt="nginx welcome page" src="https://github.com/user-attachments/assets/f088f0b8-ba86-4104-b1ea-af3c705cb035" />



## 5. Purchasing and Configuring My Custom Domain Through Cloudflare

I used Cloudflare to purchase and manage my custom domain:

**mohamedomar.dev**

I opened the Cloudflare dashboard and navigated to the domain's DNS settings.

I created the following DNS records:

| Type | Name | Content |
|---|---|---|
| A | @ | My EC2 public IPv4 address |
| A | www | My EC2 public IPv4 address |

The root domain record connected `mohamedomar.dev` to my EC2 instance, while the `www` record connected `www.mohamedomar.dev` to the same server.

I initially kept the DNS records set to DNS Only while testing the direct connection to my EC2 instance.

<img width="1920" height="877" alt="dns final record" src="https://github.com/user-attachments/assets/da1b3959-f3ab-475c-b071-6afe64dde422" />



## 6. Testing the Domain and Troubleshooting Connectivity

After configuring DNS, I tested whether my custom domain resolved to the EC2 public IP address.

I used:

```bash
nslookup mohamedomar.dev
```

I also tested the HTTP response:

```bash
curl -I http://mohamedomar.dev
```

The domain successfully returned an HTTP 200 OK response, confirming that Nginx was serving requests through my custom domain.

I also checked the Nginx listening ports:

```bash
sudo ss -tlnp
```

This helped me identify that Nginx was listening on port 80 but not yet on port 443. This meant that HTTP was working, but HTTPS had not yet been configured.

## 7. Enabling HTTPS with Let's Encrypt and Certbot

To secure the website, I installed Certbot and its Nginx plugin:

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

I then requested an SSL/TLS certificate for both my root domain and the www subdomain:

```bash
sudo certbot --nginx -d mohamedomar.dev -d www.mohamedomar.dev
```

Certbot successfully:
- Verified ownership of both domain names
- Obtained a certificate from Let's Encrypt
- Installed the certificate into the Nginx configuration
- Configured HTTPS support
- Set up automatic certificate renewal

The certificate covers both `mohamedomar.dev` and `www.mohamedomar.dev`.

<img width="1317" height="682" alt="image" src="https://github.com/user-attachments/assets/1a0cebd4-459b-4a98-8766-ecb6f90fd1d6" />


## 8. Configuring Cloudflare SSL/TLS

After enabling HTTPS on my EC2 instance, I reviewed the SSL/TLS settings in Cloudflare.

I selected **Full (strict)** as the SSL/TLS encryption mode. This allows Cloudflare to establish an encrypted connection with my origin server and verify its certificate.


## 9. Customising the Nginx Website

Once the domain and HTTPS configuration were working, I customised the default Nginx homepage.

I edited the website's HTML file:

```bash
sudo nano /var/www/html/index.html
```

I replaced the default Nginx welcome page with my own HTML and CSS.

The customised homepage introduces me and describes the technologies I used to build and deploy the website.

After saving the changes, I tested the Nginx configuration:

```bash
sudo nginx -t
```

I then reloaded Nginx:

```bash
sudo systemctl reload nginx
```

This allowed me to publish the updated homepage without restarting the entire server.


## 10. SSL Certificate Renewal

Let's Encrypt certificates have a limited validity period, so I checked that automatic renewal was configured.

I tested the renewal process using:

```bash
sudo certbot renew --dry-run
```

This allowed me to verify that Certbot can renew the certificate successfully before its expiry date.

## 11. Final Result

I successfully deployed my personal website using an AWS EC2 instance and Nginx, connected it to my custom Cloudflare domain, and enabled HTTPS using Let's Encrypt and Certbot.

The completed project demonstrated how I could:

- Provision and manage a cloud-based Linux server
- Configure AWS EC2 security groups
- Connect to a remote Ubuntu server using SSH
- Install and manage Nginx
- Configure DNS records through Cloudflare
- Connect a custom domain to an EC2 public IP address
- Obtain and install an SSL/TLS certificate
- Configure HTTPS and Cloudflare SSL/TLS settings
- Customise and publish a website using HTML and CSS

## 12. What I Learned

Through this project, I developed practical experience in deploying and managing a website using cloud infrastructure.

I gained a better understanding of how DNS connects a domain to a server, how Nginx handles incoming web requests, and how SSL/TLS certificates secure communication between a website and its visitors.

I also learned how different services, including AWS, Cloudflare, Ubuntu, Nginx and Let's Encrypt, work together to make a website publicly accessible and secure.

**Project status:** Website deployed and HTTPS certificate successfully installed.

**Live URL:** https://mohamedomar.dev
