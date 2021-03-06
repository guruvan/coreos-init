CoreOS Initialization
=====================

This is a script for preparing CoreOS systems automatically. It features the following:

* It sets up a basic environment, adding various environment files we use, primarily for DNS 
* It ensures that you have eth0 and docker0 setup, and ready for use, writing addresses to .env files
* It gets a base hostname and domain from AWS metadata
* It adds ip address info to the hostname so you can tell your cluster members apart
* It prepares a resolv.conf for use after SkyDNS is up and running on the local host. 
* It installs [fig](http://www.fig.sh/install.html)
* It installs [docker-sdlc](https://github.com/harbur/docker-sdlc)
* It installs [docker-enter](https://github.com/jpetazzo/nsenter/blob/master/docker-enter)





To use it, directly on new CoreOS instances:

<pre>
\curl -sSL https://raw.githubusercontent.com/guruvan/coreos-init/master/install | sudo bash
</pre>

Or, use the following unit from your cloudconfig:
```
    - name: install-coreos-init.service
      runtime: no
      command: start
      content: |
        [Unit]
        Description=Install from github.com/guruvan/coreos-init
        After=docker.service flanneld.service
        Requires=docker.service flanneld.service
        ConditionPathExists=!/tmp/installed.coreos-init
        [Service]
        User=root
        Type=oneshot
        RemainAfterExit=yes
        TimeoutStartSec=900
        ExecStart=/bin/sh -c "curl -ksSL https://raw.githubusercontent.com/guruvan/coreos-init/master/install |/bin/bash -x "
        ExecStop=/bin/sh -c "rm /opt/tmp/hostname.set"
```


Output will be:

<pre>
Installing fig............ [ OK ]
Installing docker-sdlc.... [ OK ]
Installing docker-enter... [ OK ]
Installing btrfs-swapon... [ OK ]
Creating swapfile......... [ OK ]
</pre>

* If a file already exists the specific installation step is skipped.
* If there is already a `/swapfile` the swap creation step is skipped.
* If docker-sdlc is already cloned, it will pull instead of clone.

You'll then be able to see the following:

Fig
---

<pre>
core@demo ~ $ fig 
Punctual, lightweight development environments using Docker.

Usage:
  fig [options] [COMMAND] [ARGS...]
  fig -h|--help

Options:
  --verbose                 Show more output
  --version                 Print version and exit
  -f, --file FILE           Specify an alternate fig file (default: fig.yml)
  -p, --project-name NAME   Specify an alternate project name (default: directory name)

Commands:
  build     Build or rebuild services
  help      Get help on a command
  kill      Kill containers
  logs      View output from containers
  ps        List containers
  rm        Remove stopped containers
  run       Run a one-off command
  scale     Set number of containers for a service
  start     Start services
  stop      Stop services
  up        Create and start containers
</pre>

docker-sdlc
-----------

docker-sdlc is a collection of fig files that compose an SDLC environment.

docker-enter
------------

Docker-enter is a wrapper script that helps to nsenter inside containers. It uses as parameter the container id or name to identify the namespace.

Note: Remember prefix `sudo` on docker-enter commands as it needs to run as root to get the necessary privileges.

<pre>
core@demo ~ $ docker run --name jenkins -d -P quay.io/harbur/jenkins
fdf7ff06b4f1d44e1558506c70b9a325d4c0595f68637c883b22968968f9c87f
</pre>

<pre>
core@demo ~ $ sudo docker-enter jenkins
root@fdf7ff06b4f1:~# 
</pre>

Swap
----

<pre>
core@demo ~ $ swapon -s
Filename				Type		Size	Used	Priority
/dev/loop2                             	partition	2097148	69420	-1
</pre>

<pre>
core@demo ~ $ free -m 
             total       used       free     shared    buffers     cached
Mem:           494        487          6          0          0         51
-/+ buffers/cache:        436         58
Swap:         2047         67       1980
</pre>

