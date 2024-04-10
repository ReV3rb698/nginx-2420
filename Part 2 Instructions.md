Assignment 3 Part 2
Caleb Campbell


Creating a reverse proxy server
=======

NOTE: This is a tutorial using the Arch Linux distribution
------
To check which disribution you are currently using, you can run the command:
`cat /etc/os-release`
This will print out the information contained within the os-release file contained within the configuration folder "/etc"
You will want to make sure that the first line *"NAME"* has the variable *"Arch Linux"*
An example of an os-release file would be:
```
NAME="Arch Linux"
PRETTY_NAME="Arch Linux"
ID=arch
BUILD_ID=rolling
ANSI_COLOR="38;2;23;147;209"
HOME_URL="https://archlinux.org/"
DOCUMENTATION_URL="https://wiki.archlinux.org/"
SUPPORT_URL="https://bbs.archlinux.org/"
BUG_REPORT_URL="https://gitlab.archlinux.org/groups/archlinux/-/issues"
PRIVACY_POLICY_URL="https://terms.archlinux.org/docs/privacy-policy/"
LOGO=archlinux-logo
```
As seen above, the os-release file contains in the first line `NAME="Arch Linux"`

Step 1
======
Installing the UFW package
------
You will want to install the ufw package using the pacman package manager using the command:

`sudo pacman -Sy ufw`

This will install the Uncomplicated Firewall package

Incase you haven't already you should update your system with the command:

`sudo pacman -Syu` 

This will update everything, and be sure to reboot the system with:

`sudo reboot`

Once you have rebooted you can proceed to step 2

Step 2
=======
Configuring the UFW
------
We will want to configure the firewall to deny outgoing traffic except for HTTP requests which go through port 80. As well as enable incoming traffic through port 22 for SSH users.

To do this we will run the following commands in order:
`sudo ufw default deny incoming`
`sudo ufw default allow outgoing`
`sudo ufw allow from 203.0.113.0/24 to any port 22`
`sudo ufw deny from 203.1.113.4`
`ufw limit ssh`
`sudo ufw allow ssh` #NOTE: Be sure to reenable SSH and port 22 so you can reconnect to your Linux server after reboot
`sudo ufw allow SSH`
`sudo ufw allow 22`
`sudo ufw allow http`
`sudo ufw limit ssh`

This will make sure to restrict traffic to only allow ports 22 and 80, HTTP and SSH to flow through

Once that is done we can enable the firewall:

`sudo ufw enable`

Then you will have to reboot the system again:

`sudo reboot`

Once that is complete, move on to Step 3

Step 3
=======
Setting up the Hello-Server service
------
Make sure your powershell is in the same directory as the hello-server file

Enter your linux server with the secure file transfer protocol with the command:

`sftp linux` #NOTE: This is similar to how you ssh into the server but you use sftp instead.

`put hello-server /home/YOUR_USER`

This will upload the hello-server to the Linux server in the home/USER directory

From there you can now move it to the required service folder

`sudo mv /home/YOUR_USER/hello-server /usr/local/bin/hello-server`

Make it executable:

`sudo chmod u+x /usr/local/bin/hello-server`

Next we can add the required service configuration after starting the hello-server

`sudo vim /etc/systemd/system/hello-server.service`

If vim isn't installed you can run the command:

`sudo pacman -Syu vim`

Once you have entered the configuration file "hello-server.service":

You will enter in the following text:
```
[Unit]
Description=Backend Service
After=network.target

[Service]
Type=Simple
#This is where the backend file is located.
ExecStart=/usr/local/bin/backend
Restart=always

[Install]
WantedBy=multi-user.target
```
Save and exit the vim by using the `:wq` method

Now we enable and start the hello-server.service:
`sudo systemctl start hello-server`
`sudo systemctl enable hello-server`

Now you can move onto Step 4

Step 4
=======
Reconfiguring the nginx server to proxy_pass:
-------

Enter into the previous configuration file from the Part 1 tutorial in our case it is:
`sudo vim /web/html/nginx-2420/nginx-2420/nginx-2420.conf`

You should see the following:
```
server {
    listen 8888;  # Listen on port 8888 for your HTTP requests
    server_name localhost;  # The domain is on localhost's localhost so we can access it with its IP address from our browser later 
    root /web/html/nginx-2420;  # Replace with the path to your HTML directory
    index index.html;  # Specify the default file to serve 
}

```
In here you add the following 
```
location /hey {
    proxy_pass http://127.0.0.1:8080;
}

location /echo {
    proxy_pass http://127.0.0.1:8080;
}
```
Save and exit the vim using `:wq`

After that you can test the server with the commands:
`sudo nginx -t`
`sudo systemctl restart nginx`

Start the hello-server.service:

`sudo systemctl start hello-server`

Once you see that is has started:
```
[2024-04-10T16:44:20Z INFO  actix_server::builder] starting 1 workers
[2024-04-10T16:44:20Z INFO  actix_server::server] Actix runtime found; starting in Actix runtime
```
IF you see that then you can proceed to Step 5.

Step 5
=======
Testing with Postman:
Make you use the GET method and run it as:
`YOUR_IP_ADDRESS/hey`
This should return:
`Hey there`
Then you use POST method and make sure you're using the JSON raw data method
*Refer to the example screenshot*
and Input:
`{"message": "Hello from your server"}`
and in the Body Output window you should see the same thing:
`{"message": "Hello from your server"}`

You have completed this tutorial!






