# Deploy Spring boot on Ubuntu/Nginx
Instructions for deploying a spring boot application on Ubuntu 16.04 using nginx as a reverse proxy. Setting up the spring boot project not covered. If you want to set up a quick test project you can go to
[Spring initializr](https://start.spring.io)

## Install Nginx
`apt-get update`  
`apt upgrade`  
`apt-get install nginx`
`apt-get install systemd`  

Check Nginx version to see if installation was successfull: `nginx -v`

## Install Java
`apt-get update`  
`apt-get install default-jre`  
`apt-get install default-jdk`  
Check Java version to see if installation was successfull: `java -version`

## Add a new user where the application will be running
`adduser <username>`  
`usermod -a -G sudo <username>`  

In the new user directory create a new directory called root. This is where the application jar will be added.

## Create an service to start the application

In `/etc/systemd/system/` create a `<myservice>.service` file and add the following.

```
[Unit]
Description=Spring Boot
After=syslog.target
After=network.target[Service]
User=<username>
Type=simple

[Service]
ExecStart=/usr/bin/java -jar /home/<username>/root/app.jar
Restart=always
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=<myservice>

[Install]
WantedBy=multi-user.target

```

## Create an Nginx Config file
In `/etc/nginx/sites-available/` create `<my-appname>` file and add the below setup:

```
server {
    listen 80;
    listen [::]:80;
    server_name <servername>;
    
        location / {
         proxy_pass http://localhost:8080/;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_set_header X-Forwarded-Proto $scheme;
         proxy_set_header X-Forwarded-Port $server_port;
    }

}
```
The default port for Spring boot applications are on port 8080 if you changed this be sure to update the port in the above configuration.  

After creating the file run the following command `ln -s /etc/nginx/sites-available/<my-appname> /etc/nginx/sites-enabled` to symlink the file to the sites enabled directory.  

Now run `systemctl restart nginx` to restart nginx for the above to take affect.

## Deploy
Build the spring boot jar file and upload it to: `/home/<username>/root/`  

Start the service we created to run the app by running: `systemctl start <myservice>`  

We can check the status of the application/service by running: `systemctl status <myservice>`  

If we want to stop the application at anytime we just run the command: `systemctl stop <myservice>`  

Now just go to any of the paths setup in spring boot and it should be up and running. 

## SSL/HTTPS

To run the application over HTTPS we will be installing certbot from letsencrypt.  

`apt-get update`  
`apt-get install software-properties-common`  
`add-apt-repository ppa:certbot/certbot`  
`apt-get update`  
`apt-get install python-certbot-nginx`  
`certbot --nginx`  

Follow the instructions and complete the certificate installation. 

### FYI 

All above commands are issued from root, to issue from the user just add `sudo` to the beginning of the command.  

In all the above wherevere something is in `<>` it means add your own value for this param. 