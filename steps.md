##### CONFIGURE NGINX AS A LOAD BALANCER
1. Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)
2. Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses

<img width="469" alt="etc:hosts" src="https://user-images.githubusercontent.com/51254648/158137455-3d55026b-82ac-4ae3-95a7-2bbac7e45369.png">

3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers
4. Update the instance and Install Nginx
    - sudo apt update
    - sudo apt install nginx
5. Open the default nginx configuration file
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

    comment out this line
    #include /etc/nginx/sites-enabled/*;

Restart Nginx and make sure the service is up and running
 - sudo systemctl restart nginx
 - sudo systemctl status nginx

<img width="829" alt="nginx running" src="https://user-images.githubusercontent.com/51254648/158137461-42de025b-348a-4824-bbd1-6b4a4e5e47e1.png">


## REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES
1. Register a new domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)
2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP
3. Update A record in your registrar to point to Nginx LB using Elastic IP address

<img width="982" alt="Domain record" src="https://user-images.githubusercontent.com/51254648/158137468-748c2ecd-8ffe-4e49-aea5-b104bf540569.png">

4. Check that your Web Servers can be reached from your browser using new domain name using HTTP protocol – http://thetwoade.com
5. Configure Nginx to recognize your new domain name
    - Make sure snapd service is active and running
        - sudo systemctl status snapd
    - Install certbot
         - sudo snap install --classic certbot
    - Request your certificate
        - sudo ln -s /snap/bin/certbot /usr/bin/certbot
        - sudo certbot --nginx
        
        <img width="704" alt="certificate request" src="https://user-images.githubusercontent.com/51254648/158137473-a2a56b89-c052-4207-a87e-88d632414d76.png">

    - Test secured access to your Web Solution by trying to reach https://thetwoade.com
    - Click on the padlock icon and you can see the details of the certificate issued for your website.
  
  <img width="487" alt="certificate" src="https://user-images.githubusercontent.com/51254648/158137476-f54805c9-afc0-404a-b438-26b0d7b5071e.png">

 6. Set up periodical renewal of your SSL/TLS certificate
    - By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently. You can test renewal command in dry-run mode
         - sudo certbot renew --dry-run
    - Best pracice is to have a scheduled job that to run renew command periodically. Let us configure a cronjob to run the command twice a day.
    - To do so, lets edit the crontab file with the following command:
            - crontab -e
        
    Add following line:

* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
You can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.

<img width="507" alt="crone cert renewal" src="https://user-images.githubusercontent.com/51254648/158137478-fd3fdfd2-71e6-484a-a909-7ac1f8ba3206.png">

    
