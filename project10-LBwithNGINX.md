# **LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS** #

It is also extremely important to ensure that connections to your Web solutions are secure and information is encrypted in transit – we will also cover connection
over secured HTTP (HTTPS protocol), its purpose and what is required to implement it.

When data is moving between a client (browser) and a Web Server over the Internet – it passes through multiple network devices and, if the data is not encrypted,
it can be relatively easy intercepted by someone who has access to the intermediate equipment. 
This kind of information security threat is called Man-In-The-Middle (MIMT) attack.

SSL and its newer version, TSL – is a security technology that protects connection from MITM attacks by creating an encrypted session between browser and
Web server. Here we will refer this family of cryptographic protocols as SSL/TLS – even though SSL was replaced by TLS, the term is still being widely used.

SSL/TLS uses digital certificates to identify and validate a Website. A browser reads the certificate issued by a Certificate Authority (CA)
to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.

In this project, we will register your website with LetsEnrcypt Certificate Authority, to automate certificate issuance you will use a
shell client recommended by LetsEncrypt – cetrbot.

### **Task** ###
This project consists of two parts:

  1. Configure Nginx as a Load Balancer
  2. Register a new domain name and configure secured connection using SSL/TLS certificates
  
Your target architecture will look like this:

![](nginx-lb.png)

## **CONFIGURE NGINX AS A LOAD BALANCER** ##

1. Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – 
this port is used for secured HTTPS connections)

![](open-80-443.jpg)

2. Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses

![](hosts-names.jpg)


3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

Update the instance and Install Nginx
~~~
sudo apt update -y && sudo apt upgrade -y
sodo apt install nginx
~~~

Configure Nginx LB using Web Servers’ names defined in /etc/hosts

Open the default nginx configuration file
~~~
sudo vi /etc/nginx/nginx.conf
~~~

#insert following configuration into *http* section
~~~
 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

    server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }
  
~~~
#comment out this line
~~~
#       include /etc/nginx/sites-enabled/*;
~~~

Restart Nginx, make sure the service is up and running and the enable the service to run when the server reboots
~~~
sudo systemctl restart nginx
sudo systemctl status nginx
sudo systemctl enable nginx
~~~

### **REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES** ###

1. Register a new domain name with any registrar of your choice in any domain zone
(e.g. .com, .net, .org, .edu, .info, .xyz or any other)

2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP

![](elastic-ip.jpg)

![](e-ip-attached.jpg)

3. Update A record in your registrar to point to Nginx LB using Elastic IP address

4. Check that your Web Servers can be reached from your browser using new domain name using HTTP protocol – http://devops.princessoo.com

![](domain-name-works.jpg)

5. Configure Nginx to recognize your new domain name
Update your nginx.conf with server_name www.<your-domain-name.com> instead of server_name www.domain.com

6. Install certbot and request for an SSL/TLS certificate
Make sure snapd service is active and running
~~~
sudo systemctl status snapd
~~~
Install certbot
~~~
sudo snap install --classic certbot
~~~

Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file so make sure you have updated it on step 5)

~~~
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
~~~

Test secured access to your Web Solution by trying to reach 
~~~
https://domain-name.com
~~~

![image](ssl-works1.jpg)

7. Set up periodical renewal of your SSL/TLS certificate
By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.

You can test renewal command in dry-run mode
~~~
sudo certbot renew --dry-run
~~~
Best pracice is to have a scheduled job that to run renew command periodically. Let us configure a cronjob to run the command twice a day.

To do so, lets edit the crontab file with the following command:
~~~
crontab -e
~~~
Selecy editor of choice
Add following line:
~~~
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
~~~
You can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.

**Success!**
