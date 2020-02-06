# Containers

## Why?
### using bare metal servers
- hosting servers
- using a kubernete nodes (which can be expensive...)
- expensive. 
- Many different projects on the same server...

### Virtual Machines
- can be scaled to whatever need. 
- Still have to manage hardware, os, plugins, etc. 
- Different virtual server. 
- Fork bomb's could take down the entire server for everyone. 

### Public Cloud
- Asure, AWS, GCP
- 30-40 different regions. 
- scale for you and you don't have to worry about it. 
- You need to upgrade your servers yourself. 
- The trade off it the it's cheaper that hiring a dedicated Dev OPs personel.
- still paying for an entire server...
  
### Containers
- All the resource and security features but is lightweight.
- a container is made up of chroot (change root), Namespaces and cgroups.
---

## `chroot`

- Change root
A way of "jailing" a process to a peticular part of the filesystem. This makes sure some files/users can't see certain files. 
In Docker, this makes files sercure only to the user of a specific image and no one else. 

### using bash in a new `chroot`ed directy. 
- create a new directory
  - `mkdir my-new-dir`
- Let's chroot this directory and use bash as the shell. 
  - `chroot my-new-dir bash`
  - you will get an error when executing this because when we wall off this directory from the rest of the server, bash no longer exists... let's install it! by copying it over!
  - `cp bin/bash my-new-dir/bin/` (create dir first...)
  - let's run bash now! ERROR!
  - bash has many dependencies, so if we run `ldd bin/bash` to show all of the shared library dependencies. 
  - Copy over each one to the `my-new-dir` directory so they match the current bin dir. 
  - Once done, bash should now work. do the same for `ls` & `cat`.

Note: to exit the shell, simply type `exit`.
## Namespaces
Namespacing is a way to completely close off the `chroot`s from eachother, so much so that they won't be able to see eachother's processes, get their own pid's, etc. 
- let's run a tail command in one of the chroots. 
  - ```tail -f secret.txt``` (this is a txt file inide one of the chroots.)
- next, let's open another shell inside the already open docker container. 
  - ```docker exec -it docker-host bash```
  - next, run ```ps```, or even ```ps aux``` to see all current processes. 
    - a = show processes for all users
    - u = display the process's user/owner
    - x = also show processes not attached to a terminal

Note: At this time, any user can not only view other user's processes, but also `kill` them. We want to prevent that with namespaces. 

 - update the server ```apt-get update```
 - install debootstap ```apt-get install debootstrap -y```
 - ```debootstrap --variant=minbase bionic /better-root```
 - A new directory named ```better-root``` will be present in the root that contains all the directories the current root directory has. 
 - Let's unshare create a new shell.
   - ```unshare --mount --uts --ipc --net --pid --fork --user --map-root-user chroot /better-root bash```

then from inside that new chroot
- mount -t proc none /proc
- mount -t sysfs non /sys
- mount -t tmpfs non /tmp

review:
- ```chroot``` is hiding/unsharing a filesystem
- ```Namspacing``` is hiding/unsharing capabilities

## cgroups
- control groups
  - originated at google. 
- htop is a great cli tool to monitor processes. 
    - To install: ```apt-get install -y cgroup-tools htop```
