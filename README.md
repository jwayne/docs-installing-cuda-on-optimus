# How to get OpenGL and CUDA (OpenCL) to work on a Ubuntu 12.04 laptop with Nvidia Optimus, for the complete novice
### February 15, 2014.

## Introduction

I’m running Ubuntu 12.04 LTS on a laptop with Nvidia Optimus graphics technology, meaning that my laptop uses two graphics cards.  In my case, these two cards are an Intel 3rd Gen integrated GPU and an Nvidia GeForce GT 630M.  I wanted to install OpenGL and CUDA to be able to develop graphics and GPU parallel processing applications, but since I was a complete noob, learning how to do this properly took untold hours (>8) of browsing.  Since Optimus is a very common setup for laptops with Nvidia (in fact, pretty much all recent laptops use Optimus [1]), I decided to write a quick tutorial to help keep future noobs from suffering as I did.

*Note 1: Instructions I’m not sure of are annotated with “[?]”.*

*Note 2: I actually wrote this tutorial in 2/2014 but never uploaded it until 9/2015.  Hopefully it still helps someone.  If you find anything wrong with it, feel free to shoot me a message or a pull request.*

## Installing GPU drivers

By default, your Ubuntu installation will come with the drivers for your integrated Intel controller.  These drivers are needed for any kind of GUI display [?], you’ll know you have them because you’ll have a black screen on boot if you don’t have them installed.  You can install these drivers from the terminal by running:

```
sudo apt-get update
sudo apt-get install xserver-xorg-video-intel
sudo reboot
```
(Source: [2])

These Intel drivers allow you to use your Intel controller and also install OpenGL.  However, you won’t be able to use your powerful Nvidia card, and furthermore you won’t be able to use CUDA.  To enable your Nvidia card, you should install **Bumblebee**, which includes an Nvidia driver [3].  Bumblebee lets your system run in two modes: By default, your system runs using the Intel GPU with the Nvidia driver turned off to save power; however, according to application load or user command, the Nvidia driver can be turned on to enable accelerated graphics performance.  Currently, Bumblebee seems to be the best option for Optimus laptops [4].  Bumblebee can be installed by running:

```
sudo add-apt-repository "deb http://archive.ubuntu.com/ubuntu $(lsb_release -sc) main universe restricted multiverse"
sudo add-apt-repository ppa:ubuntu-x-swat/x-updates
sudo add-apt-repository ppa:bumblebee/stable
sudo apt-get update
sudo apt-get install bumblebee bumblebee-nvidia linux-headers-generic
sudo reboot

```
(Source: [5] [6].  See [7] for several excellent explanations on how Bumblebee works.)

Now, your Nvidia card should work, and you should be ready to install OpenGL and CUDA.

# Installing OpenGL

OpenGL can usually be installed via the following:

```
sudo apt-get install libglu1-mesa-dev freeglut3-dev mesa-common-dev
```
(Source: [8])

When you run this, you may get a message beginning with `The following packages have unmet dependencies.` and ending with `E: Unable to correct problems, you have held broken packages.`.  I ran into this problem, and wasted many of my lost hours trying to find out the proper package version hacks to get around this.  I ended up with the following:
```
sudo apt-get install aptitude
sudo aptitude install libglu1-mesa-dev freeglut3-dev mesa-common-dev
```
When dialogs with options appear, select the first option that installs all of the packages you entered in the command line (for me, I selected the second option that appeared; existing packages were downgraded in this option, and this is OK, although you need to make sure you don’t ever upgrade these packages back).  Aptitude is a more powerful package manager than apt-get, and it works because it tries harder to install a package when there’s conflicting dependencies.
You can now test OpenGL using each of your graphics cards by running:
```
sudo apt-get install virtualgl
```
Now, if you run `glxgears` and then `optirun glxgears`, you should see frame rates for each of the runs, with the frame rate for the second run much higher.  It works!  To finish off, enable easy compilation of C code using OpenGL libraries by adding the following lines to your `~/.bashrc`:
```
export LD_LIBRARY_PATH=/usr/lib/nvidia-current:/usr/lib/nvidia-current-updates:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:/usr/local/cuda/lib:$LD_LIBRARY_PATH
```

# Installing CUDA

First, update the driver installed by bumblebee.  The default nvidia-current in the Ubuntu package repository points to nvidia-304, but CUDA requires at least nvidia-319:

```
sudo apt-get purge nvidia*
sudo apt-get install nvidia-319 nvidia-319-updates
sudo apt-get install bumblebee-nvidia
```
(Source: [9])

This requires updating the bumblebee config: [http://askubuntu.com/a/289680/248066](http://askubuntu.com/a/289680/248066)



Then, download the 64-bit RUN file at [https://developer.nvidia.com/cuda-downloads](https://developer.nvidia.com/cuda-downloads).  Now you install simply by executing that file…

```
sudo ./cuda_5.5.22_linux_64.run --optimus

```
Accept the license agreement (type ‘q’, and then ‘accept’), and then type ‘n’, ‘y’, ‘y’ when prompted to NOT INSTALL the included Nvidia driver, to INSTALL the CUDA toolkit, and to INSTALL the CUDA samples.

You’ll need to again add the suggested exports of `LD_LIBRARY_PATH` and `PATH` to your `~/.bashrc`.

Restart your computer.  Now cd to the CUDA samples directory, and type make.  After building, cd into `bin/x86_64/linux/release` and run:

```
optirun deviceQuery
```

This worked for me.  Good luck!

## References

[1] A few laptops allow for switching among integrated (Intel on, Nvidia off and hidden), discrete (Intel hidden, Nvidia on), and Optimus (Intel on, Nvidia on, with drivers for switching between the two). If you have this, then this tutorial will hold if you use Optimus mode; otherwise, see http://askubuntu.com/a/107746/248066 for what to do when the other modes are on.

[2] http://askubuntu.com/a/338496/248066

[3] Bumblebee comes packaged with Nvidia’s appropriate proprietary driver for your graphics card.  You can also install Bumblebee with the open-source Nouveau driver for Nvidia graphics cards, but Nouveau has been found to be slower than the proprietary driver (see http://www.phoronix.com/scan.php?page=article&item=nouveau_nvidia_win81&num=1).

[4] The best alternative is using Nvidia Prime, which is an analog to Bumblebee developed by Nvidia.  However, unlike Bumblebee, Prime keeps the Nvidia driver permanently on, draining battery life (my battery life with Bumblebee was 3:45 vs. Prime’s 2:15).  Technically, you can also install the appropriate proprietary Nvidia driver or appropriate open-source Nouveau driver for your card without Bumblebee; this lets your system always use the more powerful Nvidia card.  However, this generally isn’t really an option as OpenGL will not run in this environment (see http://askubuntu.com/a/134961/248066).

[5] http://askubuntu.com/a/227788/248066

[6] http://askubuntu.com/a/36936/248066

[7] http://askubuntu.com/questions/131506/

[8] http://stackoverflow.com/a/3927988/1232944

[9] http://askubuntu.com/questions/334902/12-04-update-and-nvidia

[10] http://askubuntu.com/a/133869/248066


## License

[![Creative Commons License](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/)

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).
