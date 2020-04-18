
# Instructions 
This document is to help me remember and relearn (and maybe help you learn) how to set-up a self-updating Ghost blog with Docker on a Vultr VPS and then connect it to a Ghost PWA (progressive web app) with Netlify. This way folks will be able to edit and publish posts anywhere they have access to the internet, and it will automatically rebuild and publish the static site.

---
### Vultr
Because I'm just starting out with a small website, and since it's only being used as the source for the PWA, I did some research and realized a $5/ month VPS will suffice to run Docker.

1.  Spin up a server by selecting Vultr's application tab/option, pick Docker, pick Ubuntu 18.04, change server size from $10/month to $5/month, and select other options that you need (backup, ssh keys, etc..).
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
### Setup Uncomplicated Firewall (UFW)
1. Check the status of the firewall
    ```
    sudo ufw status
    ```
1. Reset the firewall, it will disable it, so it's okay to type 'y' when it says `Resetting all rules to installed defaults. This may disrupt existing ssh connections. Proceed with operation (y|n)?`
    ```
    sudo ufw reset
    ```
1.
    ```
    ```
1.
    ```
    ```
1.
    ```
    ```
1.
    ```
    ```
1.


### Add A User To The Server
1. Create a new a user. Follow the prompts as it will ask you to create a password
    ```
    adduser NewUserName
    ```
1. Add your user to the sudo group 
    ```
    usermod -aG sudo,docker NewUserName
    ```
1. Verify your user is in the groups sudo and docker
    ```
    groups NewUserName
    ```
1. Disconnect from server as root
    ```
    exit
    ```
1. Login as the new user
    ```
    ssh NewUserName@123.456.78.90
    ```
### Add Public Key Authentication To The New User
#### On Your Local Machine
1. If you haven't already, generate a key on your local computer. Use the default locations and feel free to enter a password for the key or not - either way just don't share the private key kept on your computer with anyone
    ```
    ssh keygen -b 4096
    ```
1. If you already have a key, then display the public key, select it, and copy it
    ```
    cat ~/.ssh/id_rsa.pub
    ```
    It will look something like this
    ```
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDBGTO0tsVejssuaYR5R3Y/i73SppJAhme1dH7W2c47d4gOqB4izP0+fRLfvbz/tnXFz4iOP/H6eCV05hqUhF+KYRxt9Y8tVMrpDZR2l75o6+xSbUOMu6xN+uVF0T9XzKcxmzTmnV7Na5up3QM3DoSRYX/EP3utr2+zAqpJIfKPLdA74w7g56oYWI9blpnpzxkEd3edVJOivUkpZ4JoenWManvIaSdMTJXMy3MtlQhva+j9CgguyVbUkdzK9KKEuah+pFZvaugtebsU+bllPTB0nlXGIJk98Ie9ZtxuY3nCKneB+KjKiXrAvXUPCI9mWkYS/1rggpFmu3HbXBnWSUdf localuser@machine.local
    ```
#### On Your Remote Server     
1. Back to the server (logged in as NewUserName) create a new folder and set the folder permission
    ```
    mkdir ~/.ssh
    chmod 700 ~/.ssh 
    ```
1. Open the following file (if it doesn't exist it will be created when we open it)
    ```
    sudo nano ~/.ssh/authorized_keys
    ```
1. Paste the public key that you copied, and then type `CTRL-x` and then `y` and then `ENTER` - this will exit out of the nano editor, ask you to save it, and then quit.
1. You can add more public keys by simply pasting them on a new line
1. Set the file permission of authorized keys 
    ```
    sudo chmod 600 ~/.ssh/authorized_keys
    ```
1. Make sure ownership of NewUserName's home folder is set to NewUserName
    ```
    chown -R NewUserName:NewUserName /home/NewUserName/
    ```
1. Disable password authentication (this should happen after you add your public key, and set all the ownership and folder permission above). Open the SSH configuration file
    ```
    sudo nano /etc/ssh/sshd_config
    ```
    Find the following lines, uncomment them (delete the #) and make sure these values are set to the following
    ```
    PubkeyAuthentication  yes
    PasswordAuthentication  no
    ChallengeResponseAuthentication  no
    
    ```
1. Reload SSHD
    ```
    systemctl reload sshd
    ```
1. Now in another tab in iTerm (or hit command+d) try to ssh into the server using your NewUserName
    ```
    ssh NewUserName@123.456.78.90
    ```
    If you log in successfully it worked! If not, make sure the folder and file permission and ownership is set correctly. 
1. Make sure you can log in via your new username before changing this next line; which is to remove root login. So open up the SSH configuration file
    ```
    sudo nano /etc/ssh/sshd_config
    ```
    Find the following lines, uncomment them (delete the #) and make sure these values are set to the following
    ```
    PermitRootLogin no
    ```
    Now you will only be able to log in with your NewUserName and on your computer's whose public key you added to the authorized_keys file.

---

### Docker

1. After the server has been updated, check if Docker works. Install and remove the 'Hello World' docker container to make sure everything is working.
    ```
    docker run --rm hello-world
    ```
- Enable Docker to run when system boots
  ``` 
  systemctl enable docker 
  ```
- 
---


### Setup NGINX and MYSQL on Ubuntu 18.04

1. Install NGINX
    ```
    sudo apt install nginx
    ```
1. Check if NGINX was added to UFW (Uncomplicated Firewall) options
    ```
    sudo ufw app list
    ```
1. Add 'NGINX Full' to 
    ```
    sudo ufw allow 'Nginx Full'
    ```
1. Check status of UFW 
    ```
    sudo ufw status
    ```
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

