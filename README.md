# Nodejs-Deployment

This is just a small reference on how to deploy nodejs application to a production server

# GENERAL NOTES:

1- There are many ways to deploy node applications to servers, and it depends on the application requirements, the type of the server, developers preferences and knowledge, application and business constraints and needs...etc.
other ways might include the involvement of using technologies such as docker, jinkins, kubernetes..etc.

2- In this document I took Digital Ocean as an example of a server to deploy to using simple server login with ssh keys.

3- Commands that you need to use are all wrapped up between <> for example < npm install >; just use what's inside the angle brackets.

4- Whenever you are not on the root directory (@root) you need to use < sudo > before your commands to run them.

# STEPS:

# 1. Sign up for Digital Ocean

--link to officail website [here](https://www.digitalocean.com/).--

# 2. Create a droplet (Ubuntu Server) and login via ssh using the instructions below:

    1- In the authentication part choose SSH keys authentication.
    2- Create new ssh key using the command <ssh-keygen> on your cmd.
    3- Save the key in home/username/.ssh/id_rsa.
    4- Use the Command <cat .ssh/id_rsa.pub> to print the public key on the screen and copy it.
    5- Then on the Digital Ocean portal add a new SSH key, give it a name and set its value to the text that you copied in the previous step.
    6- Give the droplet a host name (name of your application) and create it.(dropletname)
    7- Copy the IP from the of the droplet from portal and use your cmd to login bu entering the command <ssh root@IP>
    8- You will get a connetion ensuring question enter yes and now you are logedin to your server and you will get the directory name of root@dropletname.

    *-- if you get login error try the following command:
    <ssh-add home/username/.ssh/id_rsa> and you will get Identity added to that location and then try to login again --*

# 3. Install Node/NPM using the following command:

    1- <curl -sL https://deb.nodesourse.com/setup_12.x> or <sudo apt install nodejs>
    2- <node --version (check for node)> || <npm --version>

# 4 Clone your repo from github:

    1- <git clnoe your-project.git>
        *-- You will be asked to login to github using your github credentials --*

# 5 Install dependancies and test app:

    1- <cd your-project>
    2- <npm install>
    3- <npm start> or <npm run prod> (//Any start command that you have specified eariler within your package.json commands)

    *-- Optionally you can stop your application using ctrl+c --*

## If you have a config file that you want to rename you can use the command <mv "the path of the original file with its extention" "the path of the new file with the new name including the extention as well>

## If you want to insert values to your config file you can use the command <nano "the name of the file with the extention included> which will open a text editor inside your cmd where you can asign your values. Then hit crtl+x then y to save

## If you get a problem with your mongo db connection chech that if it uses your local IP only, if it does change that in the settings at your mongodb host provider by adding the IP address of your droplet

## Digital Ocean (like many other servers providers) provide you with handy tools on your online portal one of them is Snapshots which make a live copy of your application at a specific moment in time, let's say before you donload a packge or set jobs on the database

# 6 Setup PM2 (Process Manager) to keep your app runing:

--link to officail website [here](https://pm2.keymetrics.io/).--

The above link is the official website for PM2 which is a powerful process manager from npm. Below are the commands to run your application using pm2:

    1- <npm i -g pm2> //install pm2
    2- <pm2 start "your application entery file which is usually index.js or server.js">
    3- <pm2 startup ubuntu> //Will restart the application with every server reboot
    3- <pm2 logs> //Will log your console activities

## PM2 has many other usful commands, which you can check on the official documentation on the link above such as pm2 status and pm2 restart server

###Now you have your application up and running; however you need to a web server (preferably nginx) to setup a reverse proxy so when you hit port 80 it forwards it to port 5000 or 3000 (or the port that you set within your application, you will also need to set a firewall so that no other port rather than 8080 could be accessed)###

# 7 Install NGINX and setup your firewall:

    1- <apt install nginx> //install nginx

_-- Configure firewall --_

    2- <ufw enable> //Enable firewale
    3- <ufw allow ssh> // Allowing port (note: you can put the port number)
    4- <ufw status> // Check ports status (whether its allowed or not)

_-- With this up to here your port won't work, if you refresh your IP with the port domain which is exactly what you want to do --_
_-- Now you can allow port 80 by using the following command --_

    5- <ufw allow http> //allow port 80
    6- <ufw allow https> // allow port 443

_-- Configure nginx --_

    7- <sudo nano /ect/nginx/sites-available/default> // which will open a file that you need to edit the server block (server { }) with the following block:

    server { server_name <yourdomain>.com www.<yourdomain>.com

    location / {
        proxy_pass http://localhost:5000; //(Again whatever your port was within your application)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cashe_bypass $http_upgrade;
    } }

    8- use crtl + x to exit and save
    9- service nginx restart // Restart nginx
    10- nginx -t // Check and see if everything works the way it should

    ### Now you can buy a domain name from any domain provider, and you can then assign it to your application droplet using Digital Ocean tools(Nteworking Tab) ###

# 8 Add SSL with LetsEncrypt using the following commands:

    1- < sude add-apt-repository ppa:certbot/certbot >
    2- < sudo apt-get update>
    3- < sudo apt-get install python-certbot-nginx >
    4- < sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com >

## This is only valid for 90 days, test the renwal process with <certbot renew --dre-run>

# Now you are all set, up and running ^^
