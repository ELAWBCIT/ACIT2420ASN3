## Starting Fresh - Debian 12 Server via Digital Ocean 
##### SSH Keys
- Prior to getting a Debian Digital Ocean Server up and running, a user will firstly need to get an SSH key set up.
- An SSH key is essentially an access credential that is written in the format of the SSH protocol. It helps to authenticate and establish an encrypted communication link between your remote machine on Digital Ocean and your client machine. 

#### Setting up SSH Keys 
- To begin, a user first needs to create a directory for their SSH keys. To do so, open up Windows Terminal and then do the following command 
```bash
mkdir .ssh 
```
- Next, do the following command. Be sure to replace the "Your Email Address" field to the email address in use with the Digital Ocean account. 
```bash
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
```bash
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
```bash
ssh -i <path-to-your-key> root@<ip of the debian machine> 
```

## Creating a New Regular User 
#### Adding Users ft Bash Login Shell 
- Now you will want to create a new user once you have connected to your Digital Ocean Debian Machine. 
	- The /bin/bash is to signify for your user that they want to have bash as their login shell. This is the best practice when creating new users. 
```bash
useradd -ms /bin/bash <user-name> 
```

#### Giving elevated privileges to do administrative tasks 
- To make your user able to perform the administrative tasks of your system, you will want to add them to the user group known as "sudo". 
	- Sudo standing for Superuser Do, grants your standard user elevated privileges briefly to run certain commands. 
```bash
usermod -aG sudo <user-name>
```
- You can check what groups your user has afterwards by doing the groups command with your designated username
```bash
groups <user-name>
```

#### Connecting via SSH 
- To make the new user able to connect to server via SSH, we need to copy the .ssh directory from the root users home directory to the new users home directory. 
```bash
cp -r /root/.ssh /home/<username>
```
- Now you want to change directory to your home 
```bash
cd /home/<username>
```
- Check if your .ssh file exists now in your home directory using the ls command
```bash
ls -al
```
- Now change the ownership of your file so you own it 
```bash
chown -R <username>:<groupname> /home/<username>/.ssh 
```
- **NOTE: Groupname should be the same as your username**
- Finally check if your permissions are correct now for your .ssh file with ls -al
```bash
ls -al /home/<username>
```
- You can finally exit your machine and test to see if you can get on. 
```bash
ssh -i <path-to-your-key> <username>@<ip of the debian machine> 
```

## Locking the Root User 
- **NOTE: Make sure you can login with your new user via SSH before doing this**
- Within your Debian machine, you can edit your SSH configuration so that the root user can no longer connect to the server via SSH. 
```bash
cd /etc/ssh
```
- You can open up the sshd_config file with vim 
```bash
sudo vim sshd_config 
```
- Edit the line in the vim file that says PermitRootLogin. Change it from "yes" to "no". 
- Once completed, use the following command to refresh 
```bash
systemctl restart ssh.service
```
- Now you shouldn't be able to connect to your Digital Ocean Debian Machine with root anymore. 
- You can always go and use root with the following command
```bash
sudo su -l root
```

## Installing nginx
- To install nginx, we can do so with the following command 
```bash 
sudo apt install nginx 
```

#### Configure nginx to serve a sample website 
- Firstly go to your /var/www directory with the cd command. 
```bash
cd /var/www
```
- Now you want to make a directory called "my-site". You will most likely need sudo
```bash
sudo mkdir my-site
```
- Now go into the my-site directory and create a new file called index.html 
```bash
sudo vim index.html 
```
- Copy and paste the following code into it. 
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420</title>
    <style>
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
        }
    </style>
</head>
<body>
    <h1>Hello, World</h1>
</body>
</html>
```
- Now remove the default files from the following file paths. You will need to unlink the default files from enabled using the `unlink` command. 
```bash
/etc/nginx/sites-enabled
/etc/nginx/sites-available
```
- Now we need to put a file called my_site.conf into your sites-available folder 
```bash
sudo vim my_site.conf
```
- Paste the following into the file and save it. 
```bash
server {
	listen 80 default_server;
	listen [::]:80 default_server;
	
	root /var/www/my_site;
	
	index index.html index.htm index.nginx-debian.html;
	
	server_name _;
	
	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}
}
```
- Now create a symbolic link between the two folders 
```bash
sudo ln -s /etc/nginx/sites-available/my_site.conf /etc/nginx/sites-enabled
```
- Now restart nginx to refresh it
```bash 
sudo systemctl status nginx
```
- You can check if your file works now by doing the following test command 
```bash
sudo nginx -t 
```
- Now to do a final check, you can do the `curl` command with your IP address and it should give you back the HTML. 
```bash 
curl <your ip here>
```
- If you don't know your IP address, you can do the following command to find it. 
```bash 
ip addr
```
