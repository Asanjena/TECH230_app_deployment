### Preparation
1. First, download the zipped folders: 'app' and 'environment'
2. Unzip the files by right clicking on folders and selecting 'extract all'
3. Make sure that you copy and paste these folders into the right location. For this 'project', mine was app_deployment

### Setting up Vagrant and provision files
1. create a vagrant file (vagrant innit)
2. create a provision file. To do this add a new file in the right location and name it provision.sh
3. Copy over the scripts from the previous vagrant and provision files, and paste them into the retrospective files. 

Your vagrant file should look like this:

![Alt text](images/First_vagrant.PNG)

Your provision file should look like this:

![Alt text](images/provision.PNG)

### Adding app folder from local machine to VM
1. To your vagrant file add a line saying:

```
config.vm.synced_folder "app", "/home/vagrant/app"
```
![Alt text](images/First_vagrant.PNG)

**note** - If you make a change locally, it will also do the change in your VM. Changes to your VM will also occur locally.

### Next steps to deploy the app
1. In git bash, cd to the right location

```
e.g., cd app_deployment
```

2. type 'ls' to make sure that the app is showing (add image)
3. In vscode, type:

```
cd environment

```

4. type 'ls' again to see if 'spec-tests' appears
5. If you have ruby installed, you can type 'gem install bundler'
6. Next, type 'bundle'
7. Follow this with rake spec.

**Note** - steps 5-7 will let you see all the dependencies that are required for the app, including what you already have and do not have (failures).



### Downloading the missing dependencies

for the sparta app, we were missing the package 'nodejs' (including the right version of it) and the package 'pm2'. The next section will go through how to add these dependencies




### Downloading nodejs

1. in git bash, type:

```
sudo apt-get install nodejs -y
```

2. Check that this has worked by typing 'rake spec' in vscode bash terminal (again only if you have ruby installed)

3. To get the right version of nodejs use the following codes, one-by-one, in git bash: 

```
sudo apt-get install python-software-properties

curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -

sudo apt-get install nodejs -y

```

4. Again, type 'rake spec' into vscode. In this case, you should only see 1 more failure left (which is to install pm2)




### Download pm2

1. in bash, type 'install pm2' (this is the process manager)
2. Follow this with:

```
sudo npm install pm2 -g 

(the -g means make it available globally)

```

3. Do final 'rake spec' in visual code. You should expect to see no more failures

4. Type 'ls' in bash to see if 'app' is there
5. If so, type:

```
 'cd app'
```
6. follow this with:

```
'npm install' (this is the pack manager)
```

7. Finally, type:

```
 node app.js
```

To test that you app has been deployed, in a web browser, type in your ip address, followed by the port number that is displayed in git bash 

![example of port number](images/port.PNG)
![Alt text](images/web.PNG)


### Adding to our provision.sh to get the app dependencies installed prior before entering VM

To your provision script, add the following lines:

```
sudo apt-get install python-software-properties -y 

curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -

sudo apt-get install nodejs -y

sudo npm install pm2 -g

cd /home/vagrant/app 

npm install

node app.js
```

This will automate the previous steps of downloading nodejs (and the right version of it), pm2, and npm. But as a DevOps engineer, you should always do everything step-by-step before you automate a process. 


This is how your provision.sh file should look like at the end:
![Updated provision.sh](images/updated_provisoning.PNG)




### Adding two VMs
1. Create  new folder e.g., tech230_multimachine. In this folder, copy across the vagrant and provision.sh files that we created previously. We will be making changes to these two files to create a VM for 'app' and a VM for 'db' (data base). You should also copy across the app and environment folders that we downloaded previously.

2. Alter your vagrant file so that it looks like this:

![Alt text](../tech230_multimachine/images/Vagrant.PNG)

**note** make sure that each 'end' is in the right places for their associated 'do' (as seen in the image above).

4. Your provision.sh should look like this:

![Alt text](../tech230_multimachine/images/provision.PNG)

As illustrated in the above image, comment out the last few lines so that they do not run. 

5. In a VSCode bash terminal, locate to where you saved your new folder (tech230_multimachine). 
Next type 'vagrant up'. This will start creating both of your VMs and may tke a few minutes. 

6. Open two seperate git bash terminals, and locate to tech230_multimachine on both. One bash terminal will be used for the app VM and the other will be for the db VM. 

7. In one terminal, type 'vagrant ssh app'. This terminal will be for the app VM. In the other terminal, type 'vagrant ssh db'. This will be for our db VM. 


### DB Bash Terminal and setting up MongoDB

1. Type Vagrant ssh db

2. Then upfate and upgrade the VM with any packages it finds and needs:

```
sudo apt-get update -y
```
Followed by:
```
sudo apt-get upgrade -y
```

Once this has been completed, we need to download a key for MongoDB. Type:

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D68FA50FEA312927
```
![Alt text](images/1_db.PNG)



4. Follow this with:

```
echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
```
![Alt text](images/2_db.PNG)

This will read and set where we install MongoDB from

5. To then **upgrade** and **upgrade** and make sure we have everything installed, we can use the commands:
```
sudo apt-get update -y 
sudo apt-get upgrade -y
```

This will grab all the MongoDB updates and implement them.

6. (optional) you can use 'mongod --version' to check if you have the right version

7. To install Mongo DB, type:
```
sudo apt-get install -y mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20
```
8. Next, we can start the MongoDB server using:
```
sudo systemctl start mongod
```
followed by:

```
sudo systemctl enable mongod
```

and finally:

```
sudo systemctl status mongod
```

'enable' means that multiple people can access

You terminal should then tell you that it is active:

![Alt text](images/3_db.PNG)



### Connecting the VMs 

1. First we need to go into Mongo's configuration. In nano text editor, type:
```
sudo nano /etc/mongod.conf
```
2. In the ediytor, scroll all the way to the bottom of the outputs displayed, and change it so that the bindIp says 0.0.0.0
![Alt text](images/bingIP.PNG)

This allows access from any IP address. If live/ in a production environment, you would not want this for security reasons.

3. Now that we have configured the IP adress, we need to restart and enable  the data base using

```
sudo systemctl restart mongod
```
and 
```
sudo systemctl enable mongod 
```

4. Next, we need to open our app VM bash terminal

```
vagrant ssh app
```
5. We then need to enter a global environment variable into .bashrc. This will bake it persistant/ run whenever the VM is launched and will give access to any user

```
sudo nano .bashrc
```
You should see the following:
![Alt text](images/bashrc.PNG)

Scroll all the way to the bottom of the output and add the following line:
```
export DB_HOST=mongodb://192.168.10.150:27017/posts
```

![Alt text](images/bash2.PNG)

then save and exit.


3. Run the file and add changes by typing: 
```
source .bashrc
```

4. We then need to go into our 'app' directory by using 'cd app', and run the command:

```
sudo npm install 
```
This is because we previously had commented this step out. 

5. If it seeds on its own, you will see it say on the output somewhere ' Database Seeded' and 'Database Cleared'. If it does not do this automatically, type:
```
node seeds/seed.js
```
this will put all the data from mongodb into the app

7. Then start the app by typing:
```
node app.js
```

8. if the port e.g. port 3000 is in use, use ps aux, find the line that shows where node app. js is and eneter the number frim the second coloumn after 'sudo kill' e.g. 'sudo kill 22378'. Then use 'node app.js' again to start it. 

![Alt text](images/sudokill.PNG)

9. 
To check that your posts page is working, in a web broswer type:
```
192.168.10.100.3000/posts
```

You should then see a page like this:
![Alt text](images/posts_page.PNG)


### Nginx reverse proxy research

## What are ports?
- These are numeric values that helps us to identify specific processes or servives on a computer network. They help us to know where network connections begin and end. Port numbers have specific roles and are associated with different protocols or services. Some well known ports include:
- Port 80: HTTP (Hypertext Transfer Protocol) for web traffic.
- Port 25: SMTP (Simple Mail Transfer Protocol) for email transfer.

## What is a reverse proxy?
- This is a server that sits behind the firewall of a private netwok and directs requests from clients to the right backend server(s). It receives client requests and forwards them to the appropriate backend server to fulfill those requests. The responses from the backend servers are then returned to the client through the reverse proxy.It provides  the smooth flow of network traffic between clients and servers. 
- In a regular proxy setup, the client makes a request to the proxy server, which then forwards the request to the destination server on behalf of the client. In contrast, with a reverse proxy, the client sends the request directly to the reverse proxy, which then selects the appropriate backend server to handle the request and returns the response to the client.

Reverse proxies can be used for:
- Load balancing
- Caching
- Security

![Reverse proxy diagram](images/reverse_proxy_process.png)

## Nginx configuration
By default, Nginx provides a file 
```
/etc/nginx/site-available/default
```
 that you can modify. Within the sites available directory, the two files of importance are 'default' and 'example.com' (or a similar filename)

 ### Setting up Nginx as a reverse proxy 

Make sure that your VM's are running and linked and that node js is installed

 1. In your app bash terminal, type:
 ```
 sudo nano /etc/nginx/sites-available/default
 ```

 2. Then scroll down to where it says server and location. Replace the underscore after 'server' with your ip adress. For the sparta app, this would look like:
 ```
 server_name 192.168.10.100;

```

3. Next, edit the location box so that the reverse proxy will pass the requests from the ip to the port that our app is listening on:

```
     location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    
}
```

We can also add another location block to give acces to our ports page by out MongoDB server:
```
     location /posts {
        proxy_pass http://localhost:3000/posts;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    
}
```

make sure you save and exit editor

3. To check for syntax errors, type: 
```
sudo nginx -t'
```
4. Then:
```
sudo systemctl restart nginx
```
5. Restart Nginx using:
```
 sudo systemctl restart nginx
```

6. If the connection between the app and db VMs were established correctly, navigate to your app folder (cd into app), and start the app using:
```
node app.js
```

7. To test that everything is working, type:
```
 192.168.10.100
 ```
 into a web browser, and you should see the same sparta app page. 

 ![Alt text](images/Webpage.PNG)

 To see the posts page is also working, type: 

 ```
 192.168.10.100/posts
 ```
 ![Alt text](images/posts_page.PNG)




