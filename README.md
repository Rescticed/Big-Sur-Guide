![alt text](https://i.imgur.com/RsQRjfy.png)
# Big-Sur-Guide

## Preparation 

1. Prep your machine and get ready for some work!. This takes a lot of time and dedication, so be prepared!

2. You're going to want to download VMware fusion and paragon hard disk manager from their respective websites 
VMware - https://my.vmware.com/web/vmware/downloads/details?downloadGroup=FUS-1155&productId=798&rPId=46541
Paragon Hard Disk Manager - https://www.paragon-software.com/hdm-mac/

2a. We're only using the paragon software for it's VMDK mounter, so when you mount the dmg, open the app straight from the dmg, and let it install the extra components, and voila, the Paragon VMDK mounter is in your applications folder

3. Next you're gonna want the Big Sur install app download gibMacOS from here https://github.com/corpnewt/gibMacOS and run the gibMacOS.command file and select the developer catalog with "C" and then "4" you will now see 10.16 in the list and you can download it by typing "1" 

4. after the download, go into macOS downloads folder, developer, and into the 10.16 macOS Beta folder in that. there should be an InstallAssistant.pkg. Run this, and when it's done, the Install MacOS Beta.app should be in your applications folder. 


## Getting the VM set up and installing macOS.


1. Open VMware and get it set up with a free trial. Make a new "custom" VM and set it up with the macOS 10.15 presets. 

2. Open your new VM's settings and select your 40 GB SATA drive. Drag the slider to make it 50 GB, open the advanced settings, and uncheck "split into multiple files" all other VMDKs from here on out will need this box unchecked. 

3. Hit "Add Device" and create another VMDK with 16 GB of space.

4. Mount the smaller VMDK by opening the Paragon VMDK mounter, selecting the .vmwarevm file, and selecting the smaller VMDK of the two. Ignore the popup telling you to initialize the drive.

5. Open terminal and use `diskutil list` to find the drive's identifier, such as disk3

6. run `diskutil eraseDisk JHFS+ <name of disk> GPT <location of disk>` example: `diskutil eraseDisk JHFS+ Installer GPT disk3`

7. run `sudo /Applications/Install\ macOS\ Beta/Contents/Resources/createinstallmedia —volume /Volumes/Install —nointeraction` to make the VMDK a bootable installer, then eject once done

8. Run the VM and it should boot into the installer. Open disk utility and resize the 40 GB "Macintosh HD" to 50 GB by partitioning.

9. Install macOS Big Sur as normal into the VMDK. This will take some time, so go to the "Waiting?" section

10. Once in, setup normally without iCloud, settings, or anything personal, we just want to get to the desktop.

11. As soon as you’re in the desktop, run `diskutil list` and check if there is an APFS snapshot at the end of the list as well as if `diskutil info /` returns a disk volume, like disk2s5, and not an apfs snapshot, like disk2s5s1 

12. If there is no APFS snapshot and `diskutil info /` returns a drive such as disk2s5, skip the next section. If not, follow these steps.


## Fixing APFS Snapshot


1. Boot into the installer drive again in VMware. 

(A lot of the time, it will not easily boot into the drive you want and almost never listens to the startup disk settings in either the VM or the VMware settings. If this happens, remove the drive you don’t want from the VM, keeping the file when asked, and booting fully into the one you want, then shutting down the VM, adding the drive back, selecting the option to “share with the vm that created it” as to not make accidental duplicates, and then booting again with both drives attached. It should boot into the one you want)

2. Open a terminal, type `csrutil disable` and reboot

3. When back into the installer, run `diskutil list` to find the location of the "Macintosh HD" disk, such as disk2s5

4. mount it using `diskutil mountDisk <disk identifier>` such as `diskutil mountDisk disk2s5` then run `mount -uw <volume mount point>` such as `mount -uw /Volumes/Macintosh\ HD/` to mount it as read write

5. run `/System/Library/Filesystems/apfs.fs/Contents/Resources/apfs_systemsnapshot -v <volume mount point> -r ""`

6. Then, to delete the snapshots on the disk, run `diskutil apfs listSnapshots <volume mount point>` and take note of each UUID of the snapshots

7. run `diskutil apfs deleteSnapshot <volume mount point> -uuid <uuid of snapshot>` for all snapshots on the disk. 

8. verify that there are no more snapshots by running `diskutil apfs listSnapshots <volume mount point>` it should return "No Snapshots for disk"

9. Reboot into the Big Sur desktop and make sure that running `sudo mount -uw /` returns no errors and that running `diskutil info /` returns a disk such as disk2s5 and not a snapshot such as disk2s5s1


## Making Big Sur USB 


If you didn't have an APFS snapshot, you should be here, and if you did, and you went through the steps in the section above, you should be caught up.

1. Open disk utility in Big Sur, and resize the 50 GB disk to about 1 GB larger than the space it takes up. It should be around 19.5 GB used, so resize it to 20.5-21 GB.  this will take some time, so go to the "Waiting?" section

2. Shutdown the VM and mount the larger drive with the VMDK mounter, and insert a 32 GB USB drive

(macOS will give a notification when you mount the VMDK that the drive uses features that aren't supported in this version of macOS, and Disk Utility will become buggy and unreliable. This is the reason most of this guide moving forward is the way it is. Because previous versions of macOS can't handle Big Sur drives, the conventional methods of copying data and editing partition maps are off the table. Be prepared for some jank.)

3. to prep the usb, find the disk identifier with `diskutil list`, then run `diskutil eraseDisk APFS <drive name> GPT <disk identifier>` (for proper workings, use the same drive name as the drive name in your Big Sur VM)

4. find the disk identifier of the usb and the VMDK with `diskutil list` and run `sudo dd if=/dev/r<disk identifier of VMDK> of=/dev/r<disk identifier of USB> bs=8M` such as `sudo dd if=/dev/rdisk4 of=/dev/rdisk3 bs=8M` this will take some time, so go to the "Waiting?" section

(If you want to be sure that the command is working, and are willing to do some more work, use homebrew to install "coreutils" and run the same command as before, replacing "dd" with "gdd" and ending the command with "status=progress")


## Updating EFI folder


This section should be referred to throughout, as it is a separate task than everything else and can be worked on in stages to maximize effort and time. To boot Big Sur on bare metal, you need a couple things: OC 0.6.0 compiled from source, all your kexts latest versions, compiled from source, and some minor changes to your config.plist. This covers these.

You may work on this section at any time when waiting for tasks to get done from previous steps of this guide, but do not move on to later sections until this section is done

1. OpenCore compiled from source. On a mac, this requires Xcode. 

2. run `git clone https://github.com/acidanthera/OpenCorePkg` and cd into the folder

3. run `./build_oc.tool` and wait for it to build

4. Update OC by copying the BOOT folder, Bootstrap folder, OpenCore.efi file, OpenRuntime.efi, and OpenShell.efi (if you use it). 

5. Kexts compiled from source to their most up to date versions. If a kexts you're using released its latest version already, you don't have to compile that kext. On a mac, this also requires Xcode.

6. first get a list of the kexts you're using and determine which ones you have to compile from source. It should be most of them

7. go to their github link, and copy it, then use `git clone <github link>` 

8. cd into the resulting folder and use `xcodebuild --configuration DEBUG` to compile debug version.

9. the kext should be in then build then debug folder.

10. copy the kexts to your EFI folder

(if you're using SMCBatteryManager, then for the time being, use RehabMan's ACPIBatteryManager instead)

11. Config file edits

12. NVRAM > Add > 7C436110-AB2A-4BBB-A880-FE41995C9F82 > booter-fileset-kernel - type Data - value 00

13. NVRAM > Delete > 7C436110-AB2A-4BBB-A880-FE41995C9F82 > type String - value booter-fileset-kernel


## Boot Time


1. Once your EFI folder is edited and taken care of, and all running tasks are done, make sure to test your OC build by booting back into your non-Big-Sur macOS and verify that everything works

2. After making sure that your new OpenCore configuration is working, verify that your USB has the data copied onto it correctly.

3. It's now time to try to boot into Big Sur for the first time. Make sure that the all steps before were followed and that they didn't come up with errors or weird behaviors.

4. Reboot the computer and select the external drive with Big Sur on it

5. Take care of any kernel panics or weird behaviors as best you can, using the subreddit, the discord, and the internet. Keep in mind that Big Sur is still in beta, and that OpenCore and all your Kexts may not have been fully updated to work perfectly with Big Sur.

6. When you're done fixing problems and can boot into it, with as little odd behaviors as possible, then move on to the next section.


## Final Install

1. Boot into your main macOS, and open VMware again, and open your VM's settings.

2. Remove the larger VMDK file, and actually delete the file this time, removing it from trash to make sure that unnecessary space isn't being taken up.

3. Create a new VMDK file, 50 GB in space, and add it to your VM.

4. Boot into the installer and open disk utility 

5. Rename the 50 GB drive to what you want your final install drive to be named

6. Install macOS Big Sur to your 50 GB vmdk

7. DO NOT set up the install when it's done. Verify it boots into setup, but do not set it up.

8. If you had to remove an APFS snapshot the first time, you will have to do it again. Repeat section 3 to remove it, but since you can't go into the desktop in your install, verifying that there are no APFS snapshots left on the volume will suffice.

9. Use the installer's disk utility to resize the drive to about 1 GB larger than the space it takes up once more. 

10. Shutdown the VM and mount the larger VMDK

11. Partition your drive the way you want, but add an extra (about 1-2 GB partition) at the end of the Big Sur partition.

12. use `diskutil list` to find the disk identifiers of the mounted VMDK and the partition you want Big Sur to go into (not the 1-2 GB extra partition)

13. run `sudo dd if=/dev/r<disk identifier of VMDK> of=/dev/r<disk identifier of drive partition> bs=8M` such as `sudo dd =if/dev/rdisk3 of=/dev/rdisk2 bs=8M`

(just like last time, you can optionally use "gdd" to add "status=progress" to the end to see extra info)

14. Once it's done, verify that the copying worked.

## Last Part and near the end :)

1. Reboot your computer into your Big Sur USB

2. Open disk utility and partition the disk, removing the small partition at the end of the Big Sur partition, and adding it to the end of the Big Sur partition. 

3. Verify that the APFS container has the same size as the partition it's in

4. Reboot into your Big Sur partition and set up as normal!

5. Thats it! You successfully installed Big Sur to your drive!
