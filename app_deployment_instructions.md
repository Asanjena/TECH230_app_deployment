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


### DB Bash Terminal
1. First, type:
```
sudo apt-get update -y
```
2. Follow with:
```
sudo apt-get upgrade -y
```
3. Once this has been completed, add the following line:
![Alt text](../tech230_multimachine/images/1_db.PNG)

This will add the key needed for MongoDB

4. Follow this with:

![Alt text](../tech230_multimachine/images/2_db.PNG)
This will set where we install MongoDB from

5. type 'sudo apt-get update -y', followed by 'sudo apt-get upgrade -y' again to grab all the MongoDB updates and implement them.

6. (optional) you can use 'mongod --version' to check if you have the right version

8. Type
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

![Alt text](../tech230_multimachine/images/3_db.PNG)



### Connecting the VMs 

1. First we need to go into Mongo's configuration. Type:
```
sudo nano /etc/mongod.conf
```
2. In the terminal, scroll all the way to the bottom of the outputs displayed, and change it so that the bindIp says 0.0.0.0
![Alt text](../tech230_multimachine/images/bingIP.PNG)

This allows access from any IP address. If live, you would not want this for security reasons.

3. 