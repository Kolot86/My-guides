# Simple monitoring and Telegram alerts with Uptime Kuma 
## installation 
### Packages
```bash
sudo apt update && sudo apt upgrade -y
```
### Install git
```bash
sudo apt install git 
```

### Install Nodejs and npm
```bash
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash - && \
sudo apt-get install nodejs -y && \
echo -e "\nnodejs > $(node --version).\nnpm  >>> v$(npm --version).\n"
```

### Install Uptime Kuma
   
```bash
cd $HOME
git clone https://github.com/louislam/uptime-kuma.git
cd uptime-kuma
npm run setup
```
### Install PM2 if you don't have it:

```bash
cd $HOME
sudo npm install pm2 -g && sudo pm2 install pm2-logrotate
```
### Start Server
```bash
cd $HOME/uptime-kuma
sudo pm2 start server/server.js --name uptime-kuma
```
### Browse to http://localhost:3001 after starting.

```bash
http://Your_Server_IP:3001
```
## Setup monitoring 
First, add a new monitor 

<img src="https://github.com/Kolot86/My-guides/blob/main/Pictures/uptimekuma_1%20(1).png" width="400" alt="" />

Monitor type, choose "TCP port"

<img src="https://github.com/Kolot86/My-guides/blob/main/Pictures/uptimekuma_1%20(2).png" width="400" alt="" />

As “Hostname” write IP address of your server and as a “Port”, port to which your node listens. Standard is 26656

<img src="https://github.com/Kolot86/My-guides/blob/main/Pictures/uptimekuma_1%20(6).png" width="400" alt="" />

If everything works it’s should look like this: 

<img src="https://github.com/Kolot86/My-guides/blob/main/Pictures/uptimekuma_1%20(3).png" width="400" alt="" />

## Setup telegram notification 


Go to “settings” and choose “notification” 

<img src="https://github.com/Kolot86/My-guides/blob/main/Pictures/uptimekuma_1%20(4).png" width="400" alt="" />

Choose “Setup Notification”
After you need to get a Bot token and Chat ID(or your user ID) 
Follow instructions below to do so:


| KEY | VALUE |
|---------------|-------------|
| Chat ID | Your user id you can get from [@userinfobot](https://t.me/userinfobot). The bot will only reply to messages sent from the user. All other messages are dropped and logged on the bot's console |
| Bot token  | Your telegram bot access token you can get from [@botfather](https://telegram.me/botfather). To generate new token just follow a few simple steps described [here](https://core.telegram.org/bots#6-botfather) |

Write over your Chat ID and Bot token and don’t forget to switch on “Apply on all existing monitors” 

<img src="https://github.com/Kolot86/My-guides/blob/main/Pictures/uptimekuma_1%20(5).png" width="400" alt="" />

After it’s finished, your bot should inform you if your nod is down.
Good luck! 

### Official documentation:

https://github.com/louislam/uptime-kuma
