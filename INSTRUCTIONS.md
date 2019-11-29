
# Instructions 
** This document is to help me remember and relearn (and maybe help you learn) how to set-up a self-updating Ghost blog with Docker on a Vultr VPS and then connect it to a Ghost PWA with Netlify.  


### Vultr
Because I'm just starting out with a small website, and since it's only being used as the source for the PWA, I did some research and realized a $5/ month VPS will suffice to run Docker.

- Spin up a server by selecting Vultr's application tab/option, pick Docker, pick Ubuntu 18.04, change server size from $10/month to $5/month, and select other options that you need (backup, ssh keys, etc..).
- Once it's set up, update the server 
  ```
  sudo apt-get update && sudo apt-get upgrade -y
  ```
- After the server has been updated, check if Docker works. Install and remove the 'Hello World' docker container to make sure everything is working.
  ```
  docker run --rm hello-world
  ```
- Enable Docker to run when system boots
  ``` 
  systemctl enable docker 
  ```



### Helpful Code Snippets:

- Lists all containers, even if they aren't running
  ```
  docker ps -a
  ```
- List only containers that are running
  ```
  docker ps
  ```
- Delete all stopped containers
  ```
  docker container prune
  ```
- List all images that have been downloaded
  ```
  docker image ls
  ```
- Delete all unused images
  ```
  docker image prune
  ```
  

- Use this one line code to list all packages installed without their 1st level dependencies. [Obtained from this stackexchange question](https://unix.stackexchange.com/questions/369136/list-top-level-manually-installed-packages-without-their-dependencies)
  ````
  apt-mark showmanual | sort | grep -v -F -f <(apt show $(apt-mark showmanual) 2> /dev/null | grep -e ^Depends -e ^Pre-Depends | sed 's/^Depends: //; s/^Pre-Depends: //; s/(.*)//g; s/:any//g' | tr -d ',|' | tr ' ' '\n' | grep -v ^$ | sort -u)
  ````
