TCP is the foundation of modern internet traffic.  In this tutorial, I will show you how you can optimize your server for much faster data transfer on the TCP Protocol.

### Before
To understand by how far did this improve our TCP connection, we need to have an idea of where we are at currently without the optimization.

For the tutorial today, we are using a VPS in Europe and to simulate the very worst network conditions, we chose a VPS in China. 

For testing, we are using a network testing tool called iperf3. To install on Ubuntu / Debian, use:

````sh
sudo apt-get install iperf3
````

We are going to do two tests - Single thread, and one with 8 thread.

![Single thread](https://cdn.jsdelivr.net/gh/daycat/blogimages@main/TCP-optimization/beforesinglethread.png)

The command for this is:

````sh
# on server 1:
iperf3 -s
# on server 2:
iperf3 -c <server 1 address>
````

Then, the multithread test:

![8 threaded test](https://cdn.jsdelivr.net/gh/daycat/blogimages@main/TCP-optimization/beforemultithread.png)

This can be ran with:

````sh
#on server 1:
iperf3 -s
#on server 2:
iperf3 -P <number of threads in integer> -c <server 1 address>
````

The multithreaded test above was performed with 8 threads, hence:

````sh
iperf3 -P 8 -c <114.51.4.109>
````

(Of course 114.51.4.109 is not my real ip...)

Alright! Now we have a rough idea of where our baseline speeds are. Let's get optimizing!

### Method

For this, we will be using the English port of the Nekonekocloud optimization script, which you can find on my github [here](https://github.com/daycat/tcp-optimization).

````sh
wget https://cdn.jsdelivr.net/gh/daycat/tcp-optimization@main/tcp.sh -O tcp.sh && chmod +x tcp.sh && bash tcp.sh
````

You should see this screen:
![Script menu](https://cdn.jsdelivr.net/gh/daycat/blogimages@main/TCP-optimization/tcpoptimizationmenu.png)

Once you get to this step, take a look at your System info:

If, at the end of the line. you see a number that is >= 5.x, you can skip step 1.
For example, my number is 5.10.0. Therefore, I can run step 2 straight away.

If your number is smaller than 5.x : i.e 4.9.x, you need to run step 1, which installs a version of the linux kernel that supports BBR. Please note that running step 1 can result in data loss and you **MUST** backup your data before you run it. After, reboot and run step 2.

The rest of the script's functionality isn't necessary for the TCP optimization, but I'd recommend you run 4, Resource tuning.

Lets test again!

### After

![Single thread](https://cdn.jsdelivr.net/gh/daycat/blogimages@main/TCP-optimization/aftersinglethread.png)

![Multithread](https://cdn.jsdelivr.net/gh/daycat/blogimages@main/TCP-optimization/aftermultithread.png)
