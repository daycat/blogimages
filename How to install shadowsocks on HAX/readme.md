# How to install Shadowsocks on IPv6 Only VPS

## Prerequisites
To follow this tutorial, you will need:
 - A VPS
 - A SSH tool
	 - We recommend using Termius or Finalshell. However, you can use whatever suits you the most.
- Internet connection (preferably, you should also have IPv6 if you want to make IPv6 connections to your server)
- Basic computer knowledge

## Method

### Creating a VPS

1. Go to https://hax.co.id
2. Register for a new account using your telegram account
3. On the top right corner, go to VPS -> Create VPS
![create vps](https://github.com/daycat/blogimages/raw/main/How%20to%20install%20shadowsocks%20on%20HAX/createvps.png)
5. Fill out the settings as you please. Choose "Ubuntu 20" as the Operating system, and choose "VPN server" as the purpose of the VPS

![settings](https://github.com/daycat/blogimages/raw/main/How%20to%20install%20shadowsocks%20on%20HAX/createvpssettings.png)
7. Click create VPS.

### Logging into your VPS
Please now test if you have IPv6 connectivity locally.
We recommend using https://test-ipv6.com

> If you do not have IPv6 connectivity, please follow this article:
> > https://wiki.hax.co.id/ipv6-servers/how-to-connect-to-ipv6-vps/

You need to log into your VPS account, go to VPS >> VPS info.

Here, you will find your VPS's IPv6 address.

![serverconfig](https://github.com/daycat/blogimages/raw/main/How%20to%20install%20shadowsocks%20on%20HAX/serverconfig.png)

Input the address into the terminal of your choice. The username is "root" and the password is the one you set when you created the VPS.

![serverconfig](https://github.com/daycat/blogimages/raw/main/How%20to%20install%20shadowsocks%20on%20HAX/termiusconfig.png)

> Please be reminded that you must have only letters and numbers in your password



### Updating & installing snap
````sh
sudo apt-get update && sudo apt-get install snapd && sudo snapd install core
````

This will fetch the latest updates from the Apt and Snap repos.

### Installing cloudflare WARP on your VPS

As this is an IPv6 only VPS, we must add IPv4 to access IPv4 only sites (i.e Github, etc).

We are using [fscarman](https://github.com/fscarman)'s script to do this/

````sh
wget -N https://cdn.jsdelivr.net/gh/fscarmen/warp/menu.sh && bash menu.sh
````

After, select 1, 1, 3, 3.

> For some non-kvm (LXC, OpenVZ, etc) locations, this script might not work. In this case, I suggest trying this script to mitigate the issue:
> ```` wget -N https://cdn.jsdelivr.net/gh/kkkyg/CFwarp/CFwarp.sh && bash CFwarp.sh````
> If this also does not work, please check that you have opened TUN:
> > https://wiki.hax.co.id/ipv6-servers/how-to-enable-tun-tap-on-vps/

After, check to see if you have a working IPv4 address:

````sh
ping github.com
````

You should see something like this:

![githubping](https://github.com/daycat/blogimages/raw/main/How%20to%20install%20shadowsocks%20on%20HAX/githubping.png)

### Install shadowsocks

We are using the Shadowsocks-libev version of the shadowsocks software. This has been selected as it is known to perform phenominally on lower end servers.

````sh
sudo snap install shadowsocks-libev --edge
````

### Writing a config file

After it has installed, we now need to create a config file to tell our server what to do.

First, let's generate a password to keep pesky hackers out.

````sh
openssl rand -base64 32
````

Copy this and protect it from other people. With this, they can easily connect to your server.

> You can also use your own password. However, especially if you are in China, a weak password can lead to [partitioning oracle][https://www.usenix.org/system/files/sec21summer_len.pdf#page=13] attacks against your server.

Then, let's open the config file:

````sh
vi /var/snap/shadowsocks-libev/common/etc/shadowsocks-libev/config.json
````

First, lets clean the file; press the ESC key and type:
````bash
:1,$d
````

Press enter.

Then, press 'i'

Copy this:

````sh
{
    "server":["::0","0.0.0.0"],
    "server_port":8080,
    "method":"chacha20-ietf-poly1305",
    "password":"Your Password",
    "mode":"tcp_and_udp",
    "fast_open":false
}
````

and paste, with Ctrl (Command on a mac) + V or right click (works with some terminals).

You can change the server port, method (encryption), and password to your liking. 

> For security reasons, I highly recommend you turning fast_open off.

when done, Press the ESC key, and type:
````sh
:wq
````

Done!

### Firewall (Optional, but recommended)
 It is good practice to always have a firewall on your server. For ubuntu, we are using ufw.
 
 ````sh
 sudo apt-get install ufw
 ````
 
 After installing the firewall, we need to allow the "server_port" port and the ssh port (port 22/tcp) through the firewall:
 
 ````sh
 sudo ufw allow 22/tcp
 sudo ufw allow 8080
 ````
 
Replace 8080 with whatever port you have used in the last step.
Now, enable the firewall:

````sh
sudo ufw enable
````

### Starting the server
Congratulations for reaching this step! You are now ready to start your server:

````sh
sudo systemctl start snap.shadowsocks-libev.ss-server-daemon.service
````

Then, to make sure your server runs after a reboot, use the following command:

````sh
sudo systemctl start snap.shadowsocks-libev.ss-server-daemon.service
````

There! You did it! 

