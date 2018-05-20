# Beginner's guide to building a Pi-hole

***Block ads and trackers on every device and regain control of your home network!***

This guide is in the works, but should currently contain all the information you need to get up and running. 

## A Brief FAQ

### What is a Pi-hole, and why would anyone want one?

TODO add image here

what is a pihole

what is a raspberry pi

### How difficult is this? Do I need to be a computer/network/linux/hardware expert?

Building a pi-hole is super-easy, relatively cheap, and requires no special tools or hardware skills! Some basic command line experience could be helpful, but even if you've never touched a Unix machine before you should be able to get yourself up and running without any issues. The pi-hole community is huge, with tens of thousands of people on the [SubReddit](https://www.reddit.com/r/pihole/) and [Discourse](https://discourse.pi-hole.net/), and there are loads of up-to-date guides and instructions online in case you get stuck. This guide will cover all the basics as well as some of the bells and whistles.

### How does a DNS sinkhole work, anyway?

TODO

## The Quick Summary

I've tried to include a lot of information in the guide below, but if you're interested in the tl;dr, it boils down to just a few easy steps:

1. Get a Raspberry Pi with the appropriate cables
2. Install Raspbian/NOOBS (Raspberry Pi Linux) on a microSD card and log into the pi
3. Install pi-hole on the raspberry pi (one command and a quick setup wizard)
4. Configure your router to use the Pi-hole for DNS queries
5. Add the appropriate blacklists/whitelists to the Pi-hole to block ads

## The Parts

Below I've outlined the parts you'll need to set up your pi-hole. I've also put in an 'optional' section in case you want to go a bit further and add a case, screen, etc. 

#### Bare minimum

**Raspberry Pi**

- There are a few models of Raspberry Pi out there, most of which will work for this project. 

**Power supply cable**

- Many phone chargers or microUSB cables will work in a pinch, but most don't put out enough juice for the Pi to operate stably long-term. You'll want to make sure you have a power supply that can provide sufficient voltage/current to the Pi or you could end up putting a lot of stress on the components. 

**Ethernet cable**

- Though many Raspberry Pis have wireless chips, you'll want to hardwire your pi-hole to your router to make sure it has a fast and stable connection

**MicroSD card**

- The raspberry pi has no storage, so you'll need a microSD card that's at least 4GB to hold the operating system and data. Though you can potentially get by with 4GB, I recommend 8GB-16GB so that you have space to store logs and data.
- You'll also need something that can read/write MicroSD cards. If you have a current/old Android phone, it can most likely read/write SD cards (I used my old Motorola droid). Most laptops have an SD card slot.  If not, readers are available online for a few dollars e.g. this one TODO or this one TODO.
- If you don't want to go through the trouble of creating the SD card, cards are available (e.g. TODO TODO TODO) which are pre-loaded with the raspberry pi operating system for about the same price as a card itself.

**USB mouse/keyboard and monitor**

- For the initial setup of your pi, you'll want access to a USB mouse + keyboard and a computer monitor or TV that supports HDMI. Once the Pi is set up, you won't need these anymore - you can use the web interface or ssh from another computer/phone/tablet to access and manage settings etc.

#### Optional flair

**Case**

- Adding a case makes your Raspberry pi look a lot nicer and protects it from the elements. There are loads of cheap cases ($5-20) available online at Amazon, or even cheaper from Chinese suppliers (e.g. [Monoprice](https://www.monoprice.com/), [DealExtreme](http://www.dx.com), [Alibaba](https://www.alibaba.com/), [DHGate](https://www.dhgate.com)) if you're willing to wait for shipping. 
- Make sure you pick a case that is compatible with the specific raspberry pi model you chose (e.g. most model A cases won't fit a B+). 
- If you're planning on adding a screen, make sure your case is compatible with your screen - I used a Pibow Coupe TODO case which has a low profile and doesn't get in the way of any of the pins. 

**Screen**

- If you plan to keep your Pi-hole somewhere you can see it, a screen can give you at-a-glance blocking statistics, and it also looks friggin cool. There are tons of screen options for the Pi which will work with Pi-hole, from 2-line LCD screens to full 800x480 touchscreen displays. Prices vary from $3-$60 depending on the screen.

**Stand**

- If you're keeping your pi on a desk or table and plan to install a screen, it can be nice to have a stand so you can glance at it easily (most screens don't have the best viewing angle). A simple phone stand can be had for cents online (things like [this](https://www.amazon.com/gp/product/B01HPI5AM2/) should work, and can be had for pennies on the dollar if you're willing to wait for shipping from China e.g. [Monoprice](https://www.monoprice.com/), [DealExtreme](http://www.dx.com), [Alibaba](https://www.alibaba.com/), [DHGate](https://www.dhgate.com))

## 1) Getting your Pi up and running

### A) Formatting the microSD card

Before we set up the pi-hole software, let's get the raspberry pi up and running. If you bought an SD card with Raspbian/NOOBS preloaded on it, skip to the next step

### B) Logging into your pi

Insert your microSD card into the slot on the bottom of your Pi. Using the HDMI and USB ports on the pi, connect your monitor, mouse, and keyboard. Then, connect the power cable, and the Pi will turn on. It should take you straight to the desktop after a bit of logging in. The main menu for settings etc. can be found in the top left. Network settings can be found in the top right. You can do some of the setup on wifi, but it's easier if you already have it plugged into your router via ethernet. 

### C) Configuring your pi

All Raspberry Pis come with a default username (pi) and password (raspberry). You'll want to change the password. You can do this by either opening up a **Terminal** window and typing `sudo passwd` then hitting enter, or by going to settings -> raspberry pi settings and changing it there. 

Depending on your Pi, you'll probably need to change the keyboard layout under Keyboard + Mouse Settings to English (US), otherwise the symbols will be weird.

If you want to be able to control your pi without having a keyboard/monitor connected to it once you've set it up, you'll also need to check the radio button to `enable SSH` in the 4th tab of the raspberry pi settings menu.

Finally, before you continue, find the MAC address (a unique hardware identifier for your raspberry pi) of your Pi. To do this, start a **Terminal** window

## 2) Configuring your router

For the pi-hole to work properly, it will need a static (unchanging) IP address so that your router and devices know where to find it. If you have never logged into your router, you will need to do so to change these settings. Most routers are accessible by typing either **192.168.1.1** or **192.168.0.1** into your browser's address bar. If you've never logged into your router, you may need to look up the default password, which these days is often attached to your router on a sticker somewhere. 

For the purposes of this tutorial, I'm going to assume you want to set up your pi-hole as both a **DNS** server and a **DHCP** server. **DHCP** is a system whereby a server on your network dynamically assigns IP addresses to every device on your network when it connects. On most home networks, the router serves as the DHCP server. There are a few advantages to setting up your pi-hole as a DHCP, the largest of which is that it will be able to tell which queries are coming from which device. Without setting this up, the pi-hole will see most queries as coming from your router, so you won't be able to tell which ads are being blocked on your laptop vs. your phone vs. your PS4, etc. *If you want to set up your pi-hole as a dhcp server, you will need to disable DHCP on your router. Let's do this after Step 3*

The next steps will vary based on their manufacturer, but what you want to find is something labeled along the lines of either **static IP** or **reserved IP**. This will usually be nestled in something along the lines of Advanced Settings, Setup, LAN setup, or DHCP setup. TODO

## 3) Installing and configuring Pi-hole

The folks maintaining pi-hole have made installation super easy! It will only take a couple commands to do. 

1. Open Terminal (black icon in the top menu bar) and switch to root via `sudo su`

2. Run the following command: `curl -sSL https://install.pi-hole.net | bash ` to install pi-hole

3. After installation completes, a setup wizard will appear. 

   1. First, the installation script tells you that it's **Installing packages** and that it's retrieving additional files needed for installation. Press enter to proceed through the installation until the next step
   2. The next screen asks you to **Choose An Interface** for Pi-hole to listen on. You'll want to choose `eth0` if your pihole is going to be wired to your router via an ethernet cable.
   3. The wizard now asks you to specify the **Upstream DNS Provider**. This is the service Pi-hole will use to retrieve domain name translations. I recommend the default, **Google**, or **OpenDNS**, but you can choose whichever you wish.
   4. Next, Pi-hole will prompt you to select which [internet protocols](https://www.digitalocean.com/community/tutorials/an-introduction-to-networking-terminology-interfaces-and-protocols#protocols) to filter. You'll want to make sure both IPv4 and IPv6 are checked before proceeding with enter.
   5. Pi-hole now asks if you want to use the current network settings as the **Static IP Address**. If you've configured your router properly in the previous step, you should see your chosen static/reserved IP under **IP Address**. and the router's IP address under **Gateway**. If not, you can change your IP to the desired Pi-hole IP.
   6. The next step will check if you want to enable the **web admin interface**. This is password-protected allows you to easily view traffic, whitelist and blacklist sites, and change settings from any machine on your network. I recommend keeping it **On**
   7. The Pi-hole will ask you if you want to **log queries**. Again, I recommend keeping this on, as it will allow you to keep a log of which domains your devices are trying to reach over time and enable rich tracking.
   8. This should be the end! If you've followed all the steps, your pi-hole should now be filtering any queries directed to it. 

   Now it's time to go back to the router and do some configuration

## 4) Setting up your pi-hole as a DHCP server (optional)

TODO

## 5) Configuring your blocklists

TODO

## 6) Configuring a screen to display stats (optional)

TODO
