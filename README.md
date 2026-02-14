# Tomcat Installation and Nginx Reverse Proxy

## Tomcat Installation

### EC2 instance

Login in to amazon-ec2 and **switch to root user**.
Update server.
```
yum update
```

### Install Java Corretto -17

```
yum install java-17-amazon-corretto-devel
```
### Install Tomcat

Before install tomcat. Create one ```tomcat``` user and home path ```/usr/tomcat```.

Create tomcat user
```
useradd -r -m -U -d /usr/tomcat -s /bin/bash tomcat
```
Create home directory
```
mkdir /usr/tomcat
```
Install ```tomcat 10```

```
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.52/bin/apache-tomcat-10.1.52.tar.gz
```
Untar tomcat files
```
tar -xvzf apache-tomcat-10.1.52.tar
```
Place the file in tomcat home directory ```/usr/tomcat```

Rename the file as ```tomcat_10```

```
mv apache-tomcat-10.1.52.tar tomcat_10
```

Create symlink for the tomcat_10 directory

```
ln -s /usr/tomcat/tomcat_10 /usr/tomcat/backup
```

Change ownership for tomcat file

```
chmod -R tomcat: /usr/tomcat/*
```

Create ```tomcat.service``` file 

```
vi /etc/systemd/system/tomcat.service
```

Mention ```[Unit]```,```[Service]``` and ```[Install]``` in that file.
```
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment="JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64"
Environment="CATALINA_PID=/opt/tomcat/tomcat_10/temp/tomcat.pid"
Environment="CATALINA_HOME=/opt/tomcat/tomcat_10"
Environment="CATALINA_BASE=/opt/tomcat/tomcat_10"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"
Environment="JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom"

ExecStart=/opt/tomcat/tomcat_10/startup.sh
ExecStop=/opt/tomcat/tomcat_10/bin/shutdown.sh

User=tomcat
Group=tomcat
UMask=0007
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```
Create one ```setenv.sh``` file in **/usr/tomcat/tomcat_10/bin**.

```
vi /usr/tomcat/tomcat_10/bin/setenv.sh
```
Mention the paths in that file.

```
#!/bin/bash
export JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64
export CATALINA_HOME=/usr/tomcat/tomcat_10
export CATALINA_BASE=/usr/tomcat/tomcat_10
```

Reload the daemon
```
systemctl daemon-reload
```

Start and enable tomcat

```
systemctl start tomcat
systemctl enable tomcat
```
### Accessing through browser

```
http://localhost:8080
        (or)
http://ipaddress:8080
```

**You can see the** ```Apache Tomcat/10.1.52``` **page**.

## Nginx Installation and Reverse Proxy

### Install Nginx

```
yum install nginx -y
```
Start nginx server

```
systemctl start nginx
```

Now you can access nginx through browser

```
http://localhost/
        (or)
http://ipaddress/
```
**You can see the** ```welcome to nginx``` **page**.

## Nginx Reverse Proxy

Change drive to ```/etc/nginx/conf.d```.
```
cd /etc/nginx/conf.d
```

Create ```tomcat.conf``` file

```
vi tomcat.conf
```

Place the script

```
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
Reload the nginx file

```
systemctl reload nginx
```

Start and enable the nginx

```
systemctl start nginx
systemctl enable nginx
```

 Now you can access **tomcat** via **nginx**
 
 ```
 http://localhost/
        (or)
http://ipaddress/
 ```

**Now you can see the** ```Apache Tomcat/10.1.52``` **page**.
