# Telescope development on Windows. Deployment on Ubuntu server.
## 1 Setting up development environment
### 1.1 Install Git
Download and Install Git, add Git folder to PATH
```sh
set PATH=%PATH%;C:\Program Files (x86)\Git\bin 
```
### 1.2 Node 0.10.36
Download and Install Node 0.10.36 

https://nodejs.org/dist/v0.10.36/node-v0.10.36-x86.msi

### 1.3 Meteor for Windows
Download and Install Meteor for Windows

https://install.meteor.com/windows

### 1.4 Install Meteor UP (MUP)
Open Windows Shell and execute
```sh
npm install -g mup
```
### 1.5 Installing and Running Telescope
```sh
mkdir c:\telescope
cd c:\telescope
# Clone official repository into current directory (.) 
git clone https://github.com/TelescopeJS/Telescope.git .
# Start your application
meteor 
```
(Press Ctrl-C to terminate)

Your app is available on http://localhost:3000

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
cd c:\telescope
mup init
```

#### 2.2.2 Edit deployment descriptor
Open file **c:\telescope\mup.js**

```sh
{
  // Server authentication info
  "servers": [
    {
      "host": "46.101.48.167",
      "username": "root",
      "password": "REAL_PASSWORD_HERE"
    }
  ],

  // Install MongoDB in the server, does not destroy local MongoDB on future setup
  "setupMongo": false,

  // WARNING: Node.js is required! Only skip if you already have Node.js installed on server.
  "setupNode": true,

  // WARNING: If nodeVersion omitted will setup 0.10.36 by default. Do not use v, only version number.
  "nodeVersion": "0.10.36",

  // Install PhantomJS in the server
  "setupPhantom": true,

  // Show a progress bar during the upload of the bundle to the server. 
  // Might cause an error in some rare cases if set to true, for instance in Shippable CI
  "enableUploadProgressBar": true,

  // Application name (No spaces)
  "appName": "synchronicity",

  // Location of app (local directory)
  "app": ".",

  // Configure environment
  "env": {
    "ROOT_URL": "http://46.101.48.167",
    "MONGO_URL": "mongodb://localhost:27017/synchronicity"
  },

  // Meteor Up checks if the app comes online just after the deployment
  // before mup checks that, it will wait for no. of seconds configured below
  "deployCheckWaitTime": 30
}

```

#### 2.2.4 Configure target server with mup
```sh
mup setup
```

#### 2.2.5 Now run the deploy
```sh
mup deploy
```

### Final

Access your app on http://46.101.48.167
 
