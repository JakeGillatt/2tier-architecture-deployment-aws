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

```

8. Enter the 'instance ip:3000' into your browser to check the app is running.

#
# Creating a reverse proxy server on the AWS instance


1. Use `sudo nano /etc/nginx/sites-enabled/default`
2. Remove all of the existing code and replace it with the following:
```
server {
    listen 80;
    server_name 'dynamic ip from aws site'; # app ip or db ip?

    location / {
        proxy_pass http://'dynamic ip':3000; # app ip or db ip?
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

#
# Connecting the App instance to a database instance and seeding the information:

1. Create a new instance using Ubuntu 18.04 called jake-tech221-db and add your security key tech221
2. Create a new security group, add an SSH rule with your own IP and then add a Custom TCP rule with port 27017
3. Launch the instance
4. In a new gitbash terminal, SSH into the db instance and create a provisiondb.sh file with `sudo nano provisiondb.sh`
5. Add the following code to the file:
```
#!/bin/bash
sudo apt update -y 
sudo apt upgrade -y 
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D68FA50FEA312927 
echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list 
sudo apt update -y 
sudo apt upgrade -y 
sudo apt-get install -y mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20 
sudo systemctl start mongod
```
6. Save the file
7. Change the permissions of the file with `sudo chmod +x provisiondb.sh`
8. Run the file with `./provisiondb.sh`
9. Edit the mongod.conf file with `sudo nano /etc/mongod.conf` and change the bindip to 0.0.0.0 then save the file
10. Restart mongod with `sudo systemctl restart mongod` and then `sudo systemctl enable mongod`
11. Over in the App instance, edit the .bashrc file and add `export DB_HOST=mongodb://'your DATABASE ip':27017/posts` to the bottom of the file then save the file
12. Use `source .bashrc` to reload the file
13. cd into the app with `cd app` and then run `node seeds/seed.js`
14. now run `node app.js` to launch the app and put your ip into the browser `'your ip'/posts` to get the posts page

#
# Two tier architecture diagram:

![two-tier-architecture](https://user-images.githubusercontent.com/129315605/234062497-739c912f-89ad-4f6f-93e6-d540731ef5b4.png)
