# 2 Tier Architecture on AWS

1. Launch an instance
2. SSH into your instance in  git bash
3. run `sudo apt update -y` and `sudo apt upgrade -y` to update the system
4. Install nginx with `sudo apt install nginx -y`
5. Now we need to migrate the app folder to the instance with the SCP command:

- open a new git bash terminal and cd into the .ssh folder.
- Enter the following command: `scp -i "tech221.pem" -r /c/Users/J4k3_/Documents/"sparta global"/virtualisation/app ubuntu@ec2-63-32-106-193.eu-west-1.compute.amazonaws.com:/home/ubuntu/
- The `ubuntu@ec2-63-35-178-255` part must be gathered from the connect page for the instance on AWS.

6. While the migration is running, add Custom TCP Port 3000 in your security group on AWS.
- Once the migration is completed we can install and run Node.Js
7. Run the following commands:
```
sudo apt-get install nodejs -y

curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash

sudo apt-get install nodejs -y

sudo npm install pm2 -g

cd app

npm install

node app.js
```

8. Enter the 'instance ip:3000' into your browser to check the app is running.

#
# Creating a reverse proxy server on the AWS instance


1. Use `sudo nano /etc/nginx/sites-enabled/default`
2. Remove all of the existing code and replace it with the following:
```
server {
    listen 80;
    server_name 'dynamic ip from aws site';

    location / {
        proxy_pass http://'dynamic ip':3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
2. Save the file
3. Now reload nginx with `sudo systemctl reload nginx`
4. run the app with `node app.js` and enter your ip into your browser to confirm the app page is displayed
