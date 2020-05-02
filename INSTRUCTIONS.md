
# Instructions 
This document is to help me remember and relearn (and maybe help you learn) how to set-up a self-updating Ghost blog with Docker on a Vultr VPS and then connect it to a Ghost PWA (progressive web app) with Netlify. This way folks will be able to edit and publish posts anywhere they have access to the internet, and it will automatically rebuild and publish the static site.

---
### Spin up a server - I chose Vultr
Because I'm just starting out with a small website, and since it's only being used as the source for the PWA, I did some research and realized a $5/ month VPS will suffice to run Docker.

1.  Spin up a server by selecting Vultr's application tab, pick Docker, pick Ubuntu 18.04, change server size from $10/month to $5/month, and select other options that you need (backup, ssh keys, etc..).
1.  Once it's set up, login to the server using it's IP address
    ```
    ssh root@123.456.78.90
    ```
1. Update the server 
    ```
    sudo apt update && sudo apt upgrade
    ```
1. Disable Ctrl+Alt+Delete
    ```
    sudo systemctl mask ctrl-alt-del.target
    sudo systemctl daemon-reload
    ```
---
### Add A User To The Server
1. Create a new a user. Follow the prompts as it will ask you to create a password
    ```
    adduser DinosaurDavid
    ```
1. Add your user to the sudo group 
    ```
    usermod -aG sudo,docker DinosaurDavid
    ```
1. Verify your user is in the groups sudo and docker
    ```
    groups DinosaurDavid
    ```
1. Disconnect from server as root
    ```
    exit
    ```
1. Login as the new user
    ```
    ssh DinosaurDavid@123.456.78.90
    ```
### Add Public Key Authentication To The New User
#### On Your Local Machine
1. If you don't have an ssh key already, generate a key on your local computer. Use the default locations and feel free to enter a password for the key or not - either way just don't share the private key kept on your computer with anyone
    ```
    ssh keygen -b 4096
    ```
1. If you already have a key, then display the public key, select it, and copy it
    ```
    cat ~/.ssh/id_rsa.pub
    ```
    It will look something like this
    ```
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDBGTO0tsVejssuaYR5R3Y/i73SppJAhme1dH7W2c47d4gOqB4izP0+fRLfvbz/tnXFz4iOP/H6eCV05hqUhF+KYRxt9Y8tVMrpDZR2l75o6+xSbUOMu6xN+uVF0T9XzKcxmzTmnV7Na5up3QM3DoSRYX/EP3utr2+zAqpJIfKPLdA74w7g56oYWI9blpnpzxkEd3edVJOivUkpZ4JoenWManvIaSdMTJXMy3MtlQhva+j9CgguyVbUkdzK9KKEuah+pFZvaugtebsU+bllPTB0nlXGIJk98Ie9ZtxuY3nCKneB+KjKiXrAvXUPCI9mWkYS/1rggpFmu3HbXBnWSUdf YourComputerUserName@machine.local
    ```
#### On Your Remote Server     
1. Back to the server (logged in as DinosaurDavid) create a new folder and set the folder permission
    ```
    mkdir ~/.ssh
    chmod 700 ~/.ssh 
    ```
1. Open the following file (if it doesn't exist it will be created when we open it)
    ```
    sudo nano ~/.ssh/authorized_keys
    ```
1. Paste the public key that you copied.
1. Then press `CTRL-x` and when prompted, type `y` to save the file, and then press `ENTER` which will then exit you out of the nano editor.
1. Side note: You can add more public keys by simply pasting them on a new line in the authorized_keys file
1. Set the file permission of authorized keys 
    ```
    sudo chmod 600 ~/.ssh/authorized_keys
    ```
1. Make sure ownership of DinosaurDavid's home folder is set to DinosaurDavid
    ```
    chown -R DinosaurDavid:DinosaurDavid /home/DinosaurDavid/
    ```
1. Disable password authentication (make sure this happens after you add your public key, and set all the ownership and folder permission above). Open the SSH configuration file
    ```
    sudo nano /etc/ssh/sshd_config
    ```
    Find the following lines, uncomment them (delete the #) and make sure these values are set to the following
    ```
    PubkeyAuthentication  yes
    PasswordAuthentication  no
    ChallengeResponseAuthentication  no
    
    ```
1. Then press `CTRL-x` and when prompted type `y` to save the file, and then press `ENTER` which will then exit you out of the nano editor.
1. Reload SSHD
    ```
    sudo systemctl reload sshd
    ```
1. Now in another tab in iTerm (or hit command+d) try to ssh into the server using your new user DinosaurDavid
    ```
    ssh DinosaurDavid@123.456.78.90
    ```
    If you can log in successfully, it worked! If not, make sure the permission and ownership is set correctly for the folder and file. 
1. ### Don't do this next part until you are sure you can log in with your new username!
 
1. Remove root login. First, open up the SSH configuration file
    ```
    sudo nano /etc/ssh/sshd_config
    ```
    Find the following lines, uncomment them (delete the #) and make sure the value is set to the following
    ```
    PermitRootLogin no
    ```
    Now you will only be able to log in with your DinosaurDavid user and on the computer whose whose public key you added to the authorized_keys file. Good job!
---
### Setup NGINX and UFW

1. Install NGINX
    ```
    sudo apt install nginx
    ```
1. Check the status of the firewall
    ```
    sudo ufw status
    ```
1. Reset the firewall
    ```
    sudo ufw reset
    ```
    You'll see this message:
    ```
    Resetting all rules to installed defaults. This may disrupt existing ssh connections. Proceed with operation (y|n)?
    ```
1. Type `y` and then hit enter. Don't worry, when UFW is reset, UFW gets disabled, so your ssh connection shouldn't be disrupted.
    
1. Set UFW Back to defaults
    ```
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    ```
1. Allow SSH Connections
    ```
    sudo ufw allow ssh
    ```
1. Check if NGINX was added to UFW (Uncomplicated Firewall) app list
    ```
    sudo ufw app list
    ```
    You should see something like: 
    ```
    Available applications:
    Nginx Full
    Nginx HTTP
    Nginx HTTPS
    OpenSSH
    ```
1. Add 'NGINX Full' to UFW
    ```
    sudo ufw allow 'Nginx Full'
    ```
1. Verify that SSH and Nginx Full are included 
    ```
    sudo ufw status verbose
    ```
1. Enable UFW
    ```
    sudo ufw enable
    ```
1. Verify the firewall is active!
    ```
    sudo ufw status verbose
    ```   
1. Verify that Nginx is up and running
    ```
    sudo systemctl status nginx
    ```
    You should see something like this:
    ```
    ● nginx.service - A high performance web server and a reverse proxy server
    Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
    Active: active (running) since Sun 2020-04-19 22:31:16 UTC; 1h 19min ago
    Docs: man:nginx(8)
    Main PID: 627 (nginx)
    Tasks: 2 (limit: 1108)
    CGroup: /system.slice/nginx.service
            ├─627 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
            └─628 nginx: worker process
    ```
    * If you don't see something like above, and instead get an error message like this:
        ```
        Job for nginx.service failed because the control process exited with error code.
        See "systemctl status nginx.service" and "journalctl -xe" for details.
        ```
        You most likely forget to add a `;` or added an extra character like `*` somewhere in one of your 'sites-available' configuration file (I know we haven't created those configuration files yet, so come back here to troubleshoot if this happens later). 
    
    *  Another way to check to see if there are any errors in your configuration files is to run
        ```
        sudo nginx -t
        ```
        From the `man nginx` help file, adding the `-t` flag checks the configuration file syntax and then tries to open files referenced in the configuration file.
    
    * EXAMPLE: The result after running `sudo nginx -t` will either be an error which points to the file and line number where the error occured: 
        ```
        nginx: [emerg] invalid parameter "listen" in /etc/nginx/sites-enabled/grcld.org:3
        nginx: configuration file /etc/nginx/nginx.conf test failed
        ```
        Or it will display that the test was successful:
        ```
        nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
        nginx: configuration file /etc/nginx/nginx.conf test is successful
        ```
1. Set up server blocks for different websites. We need to create a new folder for each website. Then we create each site's root folder as 'html' within it's parent website folder. Instead of typing out mkdir four separate times, we can use the -p flag to create two folders at once.
    ```
    sudo mkdir -p /var/www/gracelead.org/html
    sudo mkdir -p /var/www/grcld.org/html
    ```
1. Change the ownership of each server block to your user (or use the $USER variable if you forgot your user name):
    ```
    sudo chown -R $USER:$USER /var/www/example.com/html
    ```
1. Change the permission of each server block as well:
    ```
    sudo chmod -R 755 /var/www/example.com
    ```
1. Next we'll create a sample index.html page for each server block using nano. This allows us later to verify that we can access the server block from a browser
    ```
    sudo nano /var/www/example.com/html/index.html
    ```
    Inside, add the following sample HTML:
    ```
    <html>
        <head>
            <title>Welcome to Example.com!</title>
        </head>
        <body>
            <h1>Success!  The example.com server block is working!</h1>
        </body>
    </html>
    ```
    Then press `CTRL-x` to exit, then when prompted type `y` to save the file, and then press `ENTER` which will then exit you out of the nano editor.
1. Next we need to create a configuration file for each server block.
    ```
    sudo nano /etc/nginx/sites-available/example.com
    ```
1. Paste in the following configuration block, but replace example.com with your websites
    ```
    server {
            listen 80;
            listen [::]:80;

            root /var/www/example.com/html;
            index index.html index.htm index.nginx-debian.html;

            server_name example.com www.example.com;

            location / {
                    try_files $uri $uri/ =404;
            }
    }
    ```
    Then press `CTRL-x` to exit, then when prompted, type `y` to save the file, and then press `ENTER` which will exit you out of the nano editor.
1. Enable the configuration file for each site by creating a symbolic link to the sites-enabled directory
    ```
    sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
    ```
1. I don't completely understand this part, but to avoid hash bucket memory problems, you need to uncomment the line below in the nginx.conf file. First open the file:
    ```
    sudo nano /etc/nginx/nginx.conf
    ```
    Then remove the # symbol to uncomment the line:
    ```
    server_names_hash_bucket_size 64;
    ```

1. Next, test to make sure that there are no syntax errors in any of your Nginx files:
    ```
    sudo nginx -t
    ```
    Save and close the file when you are finished.

1. See the troubleshooting section above if you get an error, otherwise restart Nginx:
    ```
    sudo systemctl restart nginx
    ```
1. If you've already pointed your domain name to your server's IP Address, then Nginx should now be directing requests to the specific server block depending on the url that's entered. You can test this! Navigate to http://example.com (*note the http,  https isn't set up yet), and if it works correctly, you should see the index.html file you created above!

1. Congrats! Nginx is now set up with multiple server blocks! Feel free to add more if you have additional domain names. Other Nginx commands are listed below in Helpful Code Snippets
---
### Docker - Make sure it was installed correctly
1. This will install and then remove the 'Hello World' Docker container.
    ```
    docker run --rm hello-world
    ```
    You should get the following output:
    ```
    Hello from Docker!
    This message shows that your installation appears to be working correctly.

    To generate this message, Docker took the following steps:
    1. The Docker client contacted the Docker daemon.
    2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
    3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
    4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.

    To try something more ambitious, you can run an Ubuntu container with:
    $ docker run -it ubuntu bash

    Share images, automate workflows, and more with a free Docker ID:
    https://hub.docker.com/

    For more examples and ideas, visit:
    https://docs.docker.com/get-started/
    ```
1. Enable Docker to run when system boots
  ``` 
  sudo systemctl enable docker 
  ```
---


---
### Install Docker-Compose
1. Install Docker Compose using the line of code below. Use this link for the [Docker Compose GitHub repository](https://github.com/docker/compose/releases) to find the latest version and replace 1.25.5 with latest version number). And here's a link for the official [Docker docs](https://docs.docker.com/compose/install/) to install Docker Compose.
    ```
    sudo curl -L https://github.com/docker/compose/releases/download/1.25.5/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
    ```
1. Change the permission to add executable permissions to docker-compose
    ```
    sudo chmod +x /usr/local/bin/docker-compose
    ```
1. Make sure it installed correctly
    ```
    docker-compose --version
    ```
    If it installed correctly then it should display something like this:
    ```
    docker-compose version 1.25.5, build 8a1c60f6
    ```
#### If you need to delete Docker Compose
1. To Uninstall Docker Compose, simply delete the binary
    ```
    sudo rm /usr/local/bin/docker-compose
    ```
---
### Install Ghost
1. Make a Ghost Directory
    ```
    mkdir ~/ghost
    ```
1. Change directory to the ghost folder you just made
    ```
    cd ghost
    ```
1. Create docker-compose.yml file (these are the parameters by which docker will create the environment that ghost will run in)
    ```
    sudo nano docker-compose.yml
    ```
1. Paste in the following in the file you just created
    ```
    version: '3'
    services:
      ghost:
        image: ghost
        restart: always
        ports:
          - 2368:2368
        environment:
          NODE_ENV: production
          url: https://ghost.gracelead.org
        volumes:
          - /home/ghosty/ghost/content:/var/lib/ghost/content


### Helpful Code Snippets:

1. Lists all containers, even if they aren't running
    ```
    docker ps -a
    ```
1. List only containers that are running
    ```
    docker ps
    ```
1. Delete all stopped containers
    ```
    docker container prune
    ```
1. List all images that have been downloaded
    ```
    docker image ls
    ```
1. Delete all unused images
    ```
    docker image prune
    ```  
1. Use this one line code to list all packages installed on your server without their 1st level dependencies. [Obtained from this stackexchange question](https://unix.stackexchange.com/questions/369136/list-top-level-manually-installed-packages-without-their-dependencies)
    ````
    apt-mark showmanual | sort | grep -v -F -f <(apt show $(apt-mark showmanual) 2> /dev/null | grep -e ^Depends -e ^Pre-Depends | sed 's/^Depends: //; s/^Pre-Depends: //; s/(.*)//g; s/:any//g' | tr -d ',|' | tr ' ' '\n' | grep -v ^$ | sort -u)
    ````
1. Change a user password
    ```
    sudo passwd UserName
    ```
#### Nginx Comannds
1. To stop nginx
    ```    
    sudo systemctl stop nginx
    ```
1. To start nginx
    ```
    sudo systemctl start nginx
    ```
1. To restart nginx
    ```
    sudo systemctl restart nginx
    ```
1. To reload nginx (doesn't fully stop and start it)
    ```
    sudo systemctl reload nginx
    ```
1. To disable nginx from starting when the server boots
    ```
    sudo systemctl disable nginx
    ```
1. To enable nginx to start when the server boots (this is enabled by default)
    ```
    sudo systemctl enable nginx
    ```

### If you aren't using the docker image, but just installing Ghost, and want to install MySQL - this is how you do it:
1. Install MySQL
    ```
    sudo apt-get install mysql-server
    ```
1. Set a password for the default mysql user
    1. Open mysql
        ```
        sudo mysql
        ```
    1. Enter the following line but replace 'password' with your own password, and keep the quote marks, and then hit enter
        ```
        ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
        ```
    1. Exit mysql
        ```
        quit
        ```

