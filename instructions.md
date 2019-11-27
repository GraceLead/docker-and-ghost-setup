
# Instructions 
** This document is to help me remember (and maybe help you learn) how I set-up a self-updating Ghost blog with Docker on a Vultr VPS and then connect it to a Ghost PWA with Netlify.  


### Vultr
Because I'm just starting out with a small website, and since it's only being used as the source for the PWA, I did some research and realized a $5/ month VPS will suffice to run Docker. 1 CPU, 1024

- Used a one click install and installed Docker CE on Ubuntu 18.04

```
sudo apt-get update && sudo apt-get upgrade -y
```


### Helpful Code Snippets:

Use this one line code to list all packages installed without their 1st level dependencies. [Got from this stackexchange question](https://unix.stackexchange.com/questions/369136/list-top-level-manually-installed-packages-without-their-dependencies)

````
apt-mark showmanual | sort | grep -v -F -f <(apt show $(apt-mark showmanual) 2> /dev/null | grep -e ^Depends -e ^Pre-Depends | sed 's/^Depends: //; s/^Pre-Depends: //; s/(.*)//g; s/:any//g' | tr -d ',|' | tr ' ' '\n' | grep -v ^$ | sort -u)
````
