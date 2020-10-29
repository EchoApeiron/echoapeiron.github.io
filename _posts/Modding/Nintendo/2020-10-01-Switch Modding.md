---
layout: post
title: Nintendo Switch Modding
tags: nintendo switch modding storage
---
This guide will help to guide newbies through the concept of modding their Nintendo Switch or maybe even provide some refreshers for even a seasoned veteran hopefully. Though before we proceed we have to get this out of the way first. 

> Modding your Nintendo Switch enables you with the ability to do a lot of things. This can include a series of the following things:
> - Game Backups
> - Save Backups from Device NAND
> - Homebrew Applications and Games
> - Cheat Codes 
> 
> However with this being said this does mean that your device can be what is considered '**banned**'. Which means your unit **won't be able to connect to the Nintendo Store or be used to play online**. If this is something you wish to avoid I highly recommend you advert from modding your device. If you are not concerned with these risks proceed. 

**_Apologies_** but that has to be covered before proceeding. If you don't understand the risks then again don't proceed. This is a simple teether non-persistent mod. However it can compromise the use of your Nintendo Switch so don't just arbitrarily do it. **_Despite all these warnings_** if that hasn't detoured you then let's get going. 

### Required Tools and Software

While this is a very easy to execute mod. There is a few supplies and pieces of software we will require to proceed. 

#### Required Tools 

There isn't much required, but it's irresponsible to assume you might have these on hand. Before you proceed make sure you have the following: 

- RCM Jig 
- MicroSD Card 
    - At least 64GB or Greater
    - SDXC preferred, U1 as U3 isn't supported
- Card Reader 
    - For SD Card
- Windows Based PC 
    - Just makes life so easy 

#### Downloading Required Software 

Before we get too into it let's get the necessary software download. Below is the bare minimum we will need to setup: 

- [Hekate Bootloader](https://github.com/CTCaer/hekate/releases)
    - Just the archive 
- [Atmosphere Custom Firmware](https://github.com/Atmosphere-NX/Atmosphere/releases)
    - Grab both the zip archive and the `fusee-primary.bin`
- [LockPick_RCM](https://github.com/shchmue/Lockpick_RCM/releases)
    - Grab the bin file only 
- [Tegra RCM Loader](https://github.com/eliboa/TegraRcmGUI/releases)
    - I prefer the portable version but grab the installer version if you prefer 
- [MiniTool Partition Wizard Free](https://www.partitionwizard.com/free-partition-manager.html)

There are homebrew apps, sigpatches, and other sysmodules we can add. But for now we just need Hekate to do some setup work and then Atmosphere will be the CFW we will be utilizing for our modding exploits. Save these files somewhere we can find them later. 

##### Why Atmosphere and Not SX OS? 

Atmosphere is a free Custom Firmware that is widely available, open source, and fairly well suppported. While I love SX OS and the toolsets it offers from it's developers. Atmosphere is just the easiest way to go. And if you can mod a system using Atmosphere, SX OS makes it that much easier. 

### Preparing our SD Card 

The very first thing I do before starting is making sure the Switch likes the SD card. If you are using a unit where the SD card has been in use then you won't need to do anything. However if you haven't ever plugged the SD card in. Boot in the Stock Firmware and just make sure that your Switch sees it from Data Management: 

<img src="https://drive.google.com/uc?export=view&id=1WyvoGrs-uX7tmRI2xbrbxr6rv71ut7cD" alt="Switch Data Management" height="500px" />

As long as the Switch recongizes the SD card we are safe to proceed. The only other thing we can do besides validating the SD is recongized is update our Stock Firmware to the latest version. It's just another easy thing to do now and get out of the way. If you are unsure how to do this refer to the article below: 

[Nintendo Switch System Updates](https://en-americas-support.nintendo.com/app/answers/detail/a_id/22525/~/nintendo-switch-system-updates-and-change-history)

The only other thing you can do as an optional step is link a Nintendo account prior to modding. It is advised you do this prior if your account is linked do it now: 

[How to Link a Nintendo Account](https://en-americas-support.nintendo.com/app/answers/detail/a_id/22406/~/how-to-link-a-nintendo-account-to-nintendo-switch)

With the system able to recongize the SD card, Nintendo account link if desired, and with the latest firmware you can now eject the SD card. Insert it to your SD card reader and plug it into your Windows based computer. Then we will begin formatting the card as the first operation. 

#### Formatting the SD Card 

With the SD Card plugged into your computer, if you have installed MiniTool Partition Wizard. Then launch the program. Start by right clicking on the drive and deleting the entire partition: 

<img src="https://drive.google.com/uc?export=view&id=1nZvsWbsK9-Xf0h1WhowXRI0NNF9QYcRM" alt="MiniTool Delete Partition" height="500px" />

You will then want to click on the empty drive space next to it and create a new partition. Create with the settings used below: 

<img src="https://drive.google.com/uc?export=view&id=1mH2SOuwNuXXQlFRaerI1QlkPNIpXzmZ_" alt="MiniTool Create Partition" height="500px" />

The big things to note here is that we are formatting the first partition as FAT32. The other is that we are leaving about ~31GB of space `Unallocated Space After`. What this is saying is that we are leaving ~31GB of space free unpartitioned in the drive. At this point you can then create another FAT32 partition from the other remain unallocated space on the drive. Then hit apply in the bottom left corner to finish. 

#### Quickly Setup Tegra 

Before doing anything let's get Tegra setup as we will need to store our payloads somewhere safe. Simply unarchive the portable version (or install it if you choose that way), and then execute Tegra when it first opens. You will see that it is already populated with some Payloads. Simply select the top most one and hit the trash can to delete all the favorites. 

When you downloaded Atmosphere you should of gotten an archive as well as file called `fusee-primary.bin`. Let's add that payload. 

- Using Tegra next to the Select Payload text box select the little folder and navigate to fusee-primary. 
- Then in the favorites go and click the Plus sign to ensure it is added to your favorites. 
        - Note if you move the `fusee-primary.bin` file you will have to readd it to your favorites. 
        
However you will need to do this one more time with Hekate's payload. But when all is said it done it should look similar to the example below:

<img src="https://drive.google.com/uc?export=view&id=1ztW5ocnlY_ZaUk0AwWprP5GCrQGOQDlN" alt="Tegra Test Image" height="300px" /> 

#### Copy Atmosphere to the SD 

With the payload safely stashed away we need to get the custom firmware onto our SD card. Be sure on your system to grab the first partition we created. This is highly important as we left space remaining after for a reason and created the secondly partition like so for when we are doing Hekate stuff:

<img src="https://drive.google.com/uc?export=view&id=1-jMD2WXNbjR2fs6w7Re41E32fsHuCij6" alt="Windows Select Partition" height="250px" /> 

Go to where you have your Atmosphere archive downloaded and open it using your preferred archive manager (WinZip, 7-zip, etc). Then copy over the contents exactly as is to the root of the SD card: 

<img src="https://drive.google.com/uc?export=view&id=1mU9qFV_diTJjRq2__TbxAITauBu0QswQ" alt="Windows Copy Atmosphere" height="250px" /> 

#### Now Hekate and Adding LockPick_RCM

Hekate is much the same as Atmosphere. Simple enough, unarchive, copy over contents to the root of the SD card: 

<img src="https://drive.google.com/uc?export=view&id=1E5kOKlhU9qI8Zm6JBKWLnpLEvF9kVeR9" alt="Windows Copy Hekate" height="250px" /> 

Next we are going to add a payload to Hekate itself rather than Tegra. We will be adding the LockPick_RCM payload you downloaded earlier. 

- Start by opening the bootloader folder on the SD Card 
- Now open the payloads folder
- Stick the LockPick_RCM.bin file in this folder 

<img src="https://drive.google.com/uc?export=view&id=1dIFSrm5kENa8KMWYE5sw7uJYLeiD8ivh" alt="Windows Lockpick File" height="150px" /> 

This concludes the SD prep portion, now we start the real meat and potatoes.

### Hekate - Backup, EmuNAND, and Key Dumps

