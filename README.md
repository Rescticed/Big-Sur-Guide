![alt text](https://i.imgur.com/RsQRjfy.png)
# Big Surr Image Restore and Boot!


## Download DMG Image

https://drive.google.com/file/d/14IgOo4div-kyKgjalsnP7vuLo-XzfK_v/view?usp=sharing





## Restore Image to Disk 


To restore the cleaned/sanitized image, first run the first listed command (making sure to change Volume names to reflect your own accordingly), then after it finishes the check, run the second one, making sure to both run the command as superuser (sudo), and yet again change the Volume names to reflect your own accordingly:

First command: `asr imagescan --source /Volumes/YourVolumeName/Users/YourUserName/Downloads/macOS_11_ImageName.dmg`

Second command: `sudo asr restore --source /Volumes/YourVolumeName/Users/YourUserName/Downloads/macOS_11_ImageName.dmg --target /Volumes/YourVolumeNameYouWantToRestoreTo/ --erase --noverify`


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

11. Config file edits :

12. `NVRAM > Add > 7C436110-AB2A-4BBB-A880-FE41995C9F82 > booter-fileset-kernel - type Data - value 00`

13. `NVRAM > Delete > 7C436110-AB2A-4BBB-A880-FE41995C9F82 > type String - value booter-fileset-kernel`

14. Boot Arguments `-v -no_compat_check vmscgen=x -lilubetall`


## Boot Time


1. Once your EFI folder is edited and taken care of, and all running tasks are done, make sure to test your OC build by booting back into your non-Big-Sur macOS and verify that everything works

2. After making sure that your new OpenCore configuration is working, verify that your USB has the data copied onto it correctly.

3. It's now time to try to boot into Big Sur for the first time. Make sure that the all steps before were followed and that they didn't come up with errors or weird behaviors.

4. Reboot the computer and select the external drive with Big Sur on it

5. Take care of any kernel panics or weird behaviors as best you can, using the subreddit, the discord, and the internet. Keep in mind that Big Sur is still in beta, and that OpenCore and all your Kexts may not have been fully updated to work perfectly with Big Sur.

6. When you're done fixing problems and can boot into it, with as little odd behaviors as possible, then move on to the next section.



## Last Part and near the end :)

1. Reboot your computer into your Big Sur USB

2. Open disk utility and partition the disk, removing the small partition at the end of the Big Sur partition, and adding it to the end of the Big Sur partition. 

3. Verify that the APFS container has the same size as the partition it's in

4. Reboot into your Big Sur partition and set up as normal!

5. Thats it! You successfully installed Big Sur to your drive!
