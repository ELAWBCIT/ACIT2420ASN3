## Starting Fresh - Debian 12 Server via Digital Ocean 
##### SSH Keys
- Prior to getting a Debian Digital Ocean Server up and running, a user will firstly need to get an SSH key set up.
- An SSH key is essentially an access credential that is written in the format of the SSH protocol. It helps to authenticate and establish an encrypted communication link between your remote machine on Digital Ocean and your client machine. 

#### Setting up SSH Keys 
- To begin, a user first needs to create a directory for their SSH keys. To do so, open up Windows Terminal and then do the following command 
```
mkdir .ssh 
```
- Next, do the following command. Be sure to replace the "Your Email Address" field to the email address in use with the Digital Ocean account. 
```
ssh-keygen -t ed25519 -f .ssh/do-key -C "your-email-address"
```
- Now you should have a public and private key. There is an integral difference to these keys
	- Public (Ends in .pub): This is the key that is copied over to the Digital Ocean Server.
	- Private (Does not end in .pub): This key stays on your local machine.
**Note: NEVER share your SSH keys with unauthorized users or others** 

#### Adding SSH Key to your Digital Ocean account
- Login to your Digital Ocean account. Now head to settings. 
- In settings go to security menu in the console and click "Add SSH Key" button. 
- You can do the following command to get your SSH Key to put into Digital Ocean with this command 
```
Get-Content C:\Users\user-name\.ssh\do-key.pub | Set-Clipboard
```

#### Digital Ocean configurations 
- Now go to your main Digital Ocean dashboard. 
- Hit "Create" and select "Droplets". Then select the following settings:
	- Leave the server on San Francisco SFO3
	- Select Debian as the OS. 
	- Go for the $7/mo 0.010/hour option in CPU options. 
	- Leave authentication method on SSH key. 
- Once you're done, hit "Create Droplet" 
- Now you can connect to your new droplet with the following
```
ssh -i <path-to-your-key> root@<ip of the debian machine> 
```

## Creating a New Regular User 
#### Adding Users ft Bash Login Shell 
- Now you will want to create a new user once you have connected to your Digital Ocean Debian Machine. 
	- The /bin/bash is to signify for your user that they want to have bash as their login shell. This is the best practice when creating new users. 
```
useradd -ms /bin/bash <user-name> 
```

#### Giving elevated privileges to do administrative tasks 
- To make your user able to perform the administrative tasks of your system, you will want to add them to the user group known as "sudo". 
	- Sudo standing for Superuser Do, grants your standard user elevated privileges briefly to run certain commands. 
```
usermod -aG sudo <user-name>
```
- You can check what groups your user has afterwards by doing the groups command with your designated username
```
groups <user-name>
```

#### Connecting via SSH 
- To make the new user able to connect to server via SSH, we need to copy the .ssh directory from the root users home directory to the new users home directory. 
```
cp -r /root/.ssh /home/<username>
```
- Now you want to change directory to your home 
```
cd /home/<username>
```
- Check if your .ssh file exists now in your home directory using the ls command
```
ls -al
```
- Now change the ownership of your file so you own it 
```
chown -R <username>:<groupname> /home/.ssh 
```
- **NOTE: Groupname should be the same as your username**
- Finally check if your permissions are correct now for your .ssh file with ls -al
```
ls -al /home/<username>
```
- You can finally exit your machine and test to see if you can get on. 
```
ssh -i <<path-to-your-key> <username>@<ip of the debian machine> 
```

## Locking the Root User 
- **NOTE: Make sure you can login with your new user via SSH before doing this**
- Within your Debian machine, you can edit your SSH configuration so that the root user can no longer connect to the server via SSH. 
```
cd /etc/ssh
```
- You can open up the sshd_config file with vim 
```
vim sshd_config 
```
- Edit the line in the vim file that says PermitRootLogin. Change it from "yes" to "no". 
- Once completed, use the following command to refresh 
```
systemctl restart ssh.service
```
- Now you shouldn't be able to connect to your Digital Ocean Debian Machine with root anymore. 
- You can always go and use root with the following command
```
sudo su -l root
```
