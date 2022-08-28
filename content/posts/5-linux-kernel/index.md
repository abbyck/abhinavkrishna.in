---
layout: page
title: "Going through Linux Device Drivers, 3rd Edition"
date: 2022-08-24
showSummary: true
categories: ['learning']
tags: ['linux', 'kernel']
---

I was introduced to `Linux Device Drivers`
*Book by Alessandro Rubini, Greg Kroah-Hartman, and Jonathan Corbet* in a video by LiveOverflow from 2020. 
But I never got around to reading it.
This post is just me jotting down my experiences going through this book.

{{< youtube juGNPLdjLH4 >}}

### Linux Device Drivers, Third Edition
This book is freely available from [lwn.net](https://lwn.net/Kernel/LDD3/) under the Creative Commons 
"Attribution-ShareAlike" license, Version 2.0. LiveOverflow goes through
a bit of history in his video. I think I don't need to reiterate it. Oh, and BTW, sub to lwn.net if you can!

I built the kernel with the existing config from my Arch Linux install a couple of years back. And custom kernels for
building TWRP and some test ROMs for my previous android devices. But that was it. The kernel source is too scary for me :sweat_smile:

So, in other words, this is another attempt to try and understand the kernel source, or at least some parts of it.
Heard, device drivers are a good starting point? Well, the kernel *can* be thought of as a driver.

Chapter 1 is a lighter version of my Engineering Degree's [CS204](https://drive.google.com/file/d/1OlVHtCAfyOcfXQPs_0i0wmM9FRrl60Ao/view) syllabus but a bit skewed towards Linux Kernel.

Also, since LDD3 was created with Linux 2.6 in mind, some coding examples will require some modifications.
There is this repo [ldd3](https://github.com/martinezjavier/ldd3) which has examples updated to work in recent kernels.

### Chapter 2
So, the real meat starts from chapter 2 which contains a hello world kernel module!

I jumped on Vagrant to set up a temporary, quick VM. Just followed the 
[Vagrant ArchWiki](https://wiki.archlinux.org/title/Vagrant) and used `libvirt` 
as the virtualization provider. Also had to install `nfs-utils` along with the `libvirt` dependencies.


Ran `vagrant init` and edited the Vagrantfile to include Debian Bullseye box `debian/bullseye64`.
A sample config might look something like this.


```yaml
Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64"

  config.vm.provider "libvirt" do |v|
    # Adjust these as required
    v.memory = 8162
    v.cpus = 4
  end
end
```

Now, just do `vagrant up` and if it's successful, you can do `vagrant ssh` to get into the VM.
You also need to install build-essentials and appropriate kernel headers for your host(VM) 
-> ``apt install build-essential linux-headers-`uname -r` ``.

Copied over the hello.c file from [here](https://github.com/martinezjavier/ldd3/blob/master/misc-modules/hello.c) to `/vagrant/src` directory on the guest (or to `./src` directory on the host)
and added the [makefile](https://github.com/martinezjavier/ldd3/blob/master/misc-modules/Makefile). Make sure to remove
all the modules that we are not building at this step -> Only keep `hello.o` in `obj-m`.

While copying, I've also noticed this placement of braces and spaces. It's not something that I'm used to. So it might be worth looking at the [very opinionated coding style for the kernel](https://www.kernel.org/doc/html/v4.10/process/coding-style.html). 

Now if you run `make`, it should hopefully build our very very complex kernel modules without errors :)

To insert our module to the kernel, we can use insmod; `sudo insmod hello.ko`. Note the .ko extension indicating that it's a kernel object.

At this point we can inspect the kernel logs and we should see the function we defined in the 
`module_init` macro getting executed. 

`sudo tail /var/log/kern.log` ->

```shell
Aug 24 09:04:43 bullseye kernel: [ 1143.598608] hello: loading out-of-tree module taints kernel.
Aug 24 09:04:43 bullseye kernel: [ 1143.599432] hello: module verification failed: signature and/or required key missing - tainting kernel
Aug 24 09:04:43 bullseye kernel: [ 1143.600865] Hello, world
```

How exciting is this! Something we wrote is getting executed in the kernel space, not in userspace!!

To remove this module, we can run `sudo rmmod hello` and see our poor module realizing the realities of the world :P
```shell
Aug 24 09:26:00 bullseye kernel: [ 2421.287445] Goodbye, cruel world
```

Just to spice things a bit I decided to calculate the factorial of 10 from the kernel space :laughing:

```c
#include <linux/init.h>
#include <linux/module.h>
MODULE_LICENSE("Dual BSD/GPL");
static int hello_init(void)
{
printk(KERN_ALERT "Hello, world\n");
return 0;
}
static void hello_exit(void)
{
long fact = 1;
int i = 1;
while(i<=10)
        {
                fact*=i;
                i++;
        }
printk(KERN_INFO "Factorial: %ld\n", fact);
}

module_init(hello_init);
module_exit(hello_exit);
```
I'm pretty sure that I have broken 10 different kernel coding guidelines with these ^ lines.
And is probably against the conventions, but, it's fun!
```shell
Aug 28 10:04:26 bullseye kernel: [ 4727.115511] Hello, world
Aug 28 10:04:30 bullseye kernel: [ 4730.577740] Factorial: 3628800
```

I'll be back with moarr sweet kernel juice.