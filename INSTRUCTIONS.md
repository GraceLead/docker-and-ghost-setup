
# Instructions 
** This document is to help me remember and relearn (and maybe help you learn) how to set-up a self-updating Ghost blog with Docker on a Vultr VPS and then connect it to a Ghost PWA with Netlify.  

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
1. After the server has been updated, check if Docker works. Install and remove the 'Hello World' docker container to make sure everything is working.
    ```
    docker run --rm hello-world
    ```
- Enable Docker to run when system boots
  ``` 
  systemctl enable docker 
  ```
---
### Adding A User So We Don't Use Root
1. Create a new a user. Follow the prompts as it will ask you to create a password)
    ```
    adduser UserName
    ```
1. Add your user to the sudo group 
    ```
    usermod -aG sudo,docker UserName
    ```
1. Verify your user is in the groups sudo and docker
    ```
    groups UserName
    ```
1. Disconnect from server as root
    ```
    exit
    ```
1. Login as new user
    ```
    ssh UserName@123.456.78.90
    ```
---


---
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

