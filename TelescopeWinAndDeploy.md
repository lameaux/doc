# Telescope development on Windows. Deployment on Ubuntu server.

## 1 Setting up development environment
### 1.1 Install Oracle Virtual Box
Download and Install Oracle Virtual Box, add Virtual Box folder to PATH
```sh
set PATH=%PATH%;C:\Program Files\Oracle\VirtualBox 
```
### 1.2 Install Git
Download and Install Git, add Git folder to PATH
```sh
set PATH=%PATH%;C:\Program Files (x86)\Git\bin 
```
### 1.3 Vagrant
Download Vagrant from https://www.vagrantup.com/downloads.html and install Vagrant to **c:\vagrant**

Open Command Prompt as Administrator and run:
```sh
cd c:\vagrant
vagrant init precise32 http://files.vagrantup.com/precise32.box 
```
Edit **c:\vagrant\Vagrantfile** by adding the following line
```sh
config.vm.network :forwarded_port, guest: 3000, host: 3000
```
after that line
```sh
Vagrant.configure(“2”)
```

Start Vagrant box with
```sh
vagrant up
```

Connect to Vagrant with
```sh
vagrant ssh
```

### 1.4 Configure Vagrant environment
```sh
sudo apt-get update
sudo apt-get install -y python-software-properties

# mongodb
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv 7F0CEB10
echo "deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart \
dist 10gen" | sudo tee -a /etc/apt/sources.list.d/10gen.list
sudo apt-get update
sudo apt-get install -y git mongodb-10gen curl

#nodejs
cd /usr/local
sudo wget http://nodejs.org/dist/v0.12.0/node-v0.12.0-linux-x86.tar.gz
sudo tar -xvzf node-v0.12.0-linux-x86.tar.gz –-strip=1
sudo rm -f node-v0.12.0-linux-x86.tar.gz

# meteor
curl https://install.meteor.com | sudo sh
sudo npm install -g meteorite

# sshpass and mup
sudo apt-get install sshpass
sudo npm install -g mup

sudo iptables --flush
```

### 1.5 Installing and Running Telescope
```sh
mkdir ~/telescope
cd /vagrant
# Installing from official repository
git clone https://github.com/TelescopeJS/Telescope.git telescope
sudo rsync -rtvu /vagrant/telescope/ /home/vagrant/telescope/
cd ~/telescope

sudo meteor 
```
(Press Ctrl-C to terminate)

Your app is available on http://localhost:3000

### 1.6 Development process
Telescope is available in **c:\vagrant\telescope**

Running meteor in background:
```sh
cd ~/telescope
sudo meteor &
```
Press Ctrl-C to leave Telescope on background.

Updating changes made on local environment:
```sh
sudo rsync -rtvu /vagrant/telescope/ /home/vagrant/telescope/
```

## 2 Deployment

### 2.1 Configure target server

Log into your production server and execute the following commands to install MongoDB:

```sh
sudo apt-get update
sudo apt-get install -y python-software-properties
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv 7F0CEB10
echo "deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart \
dist 10gen" | sudo tee -a /etc/apt/sources.list.d/10gen.list
sudo apt-get update
sudo apt-get install -y git mongodb-10gen curl
```

More info: http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/#install-mongodb

### 2.2 Configure Deployment Descriptor

#### 2.2.1 Init project

```sh
cd /vagrant/telescope
sudo mup init
```

#### 2.2.2 Edit deployment descriptor
Open file **c:\vagrant\telescope\mup.js**

```sh
  // Server authentication info
  "servers": [
    {
      "host": "178.62.69.100",
      "username": "root",
      "password": "telescope"
      // or pem file (ssh based authentication)
      //"pem": "~/.ssh/id_rsa"
    }
  ],

  // Install MongoDB in the server, does not destroy local MongoDB on future setup
  "setupMongo": false,

  // WARNING: Node.js is required! Only skip if you already have Node.js installed on server.
  "setupNode": true,

  // WARNING: If nodeVersion omitted will setup 0.10.33 by default. Do not use v, only version number.
  "nodeVersion": "0.10.33",

  // Install PhantomJS in the server
  "setupPhantom": true,

  // Application name (No spaces)
  "appName": "telescope",

  // Location of app (local directory)
  "app": "/home/vagrant/telescope",

  // Configure environment
  "env": {
    "ROOT_URL": "http://178.62.69.100",
    "MONGO_URL": "mongodb://localhost:27017/telescope"
  },

  // Meteor Up checks if the app comes online just after the deployment
  // before mup checks that, it will wait for no. of seconds configured below
  "deployCheckWaitTime": 30
}
```

#### 2.2.4 Configure target server with mup
```sh
sudo rsync -rtvu /vagrant/telescope/ /home/vagrant/telescope/
cd /home/vagrant/telescope
sudo mup setup
```

#### 2.2.5 Now run the deploy
```sh
sudo mup deploy
```

### Final

Access your app on http://178.62.69.100

