##### CONFIGURE NGINX AS A LOAD BALANCER
1. Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)
2. Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses
img
3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers
4. Update the instance and Install Nginx
    - sudo apt update
    - sudo apt install nginx
 Open the default nginx configuration file
  - sudo vi /etc/nginx/nginx.conf
#insert following configuration into http section

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

#comment out this line
#       include /etc/nginx/sites-enabled/*;

Restart Nginx and make sure the service is up and running
 - sudo systemctl restart nginx
 - sudo systemctl status nginx
img

## REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES
1. Register a new domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)
2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP
3. Update A record in your registrar to point to Nginx LB using Elastic IP address
img
4. Check that your Web Servers can be reached from your browser using new domain name using HTTP protocol – http://thetwoade.com
5. Configure Nginx to recognize your new domain name
    - Make sure snapd service is active and running
        sudo systemctl status snapd
    - Install certbot
        sudo snap install --classic certbot
    - Request your certificate
        sudo ln -s /snap/bin/certbot /usr/bin/certbot
        sudo certbot --nginx
        img
    - Test secured access to your Web Solution by trying to reach https://thetwoade.com
    - Click on the padlock icon and you can see the details of the certificate issued for your website.
    img
 6. Set up periodical renewal of your SSL/TLS certificate
    - By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently. You can test renewal command in dry-run mode
         sudo certbot renew --dry-run
    - Best pracice is to have a scheduled job that to run renew command periodically. Let us configure a cronjob to run the command twice a day.
    - To do so, lets edit the crontab file with the following command:
            crontab -e
        Add following line:

* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
You can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.
img

    
