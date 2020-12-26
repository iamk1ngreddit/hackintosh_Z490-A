# hackintosh_Z490-A

This Guide details the successful Hackintosh build for OSX 11 / Big Sur with OpenCore 0.6.4

Special Thanks to yilmazca, without his initial Guide, this would have been 100x harder.  His Guide / Github can be found here:
https://github.com/yilmazca/intel-i9-10900K-Asus-prime-Z490A-hackintosh


<h2>Hardware:</h2>

- ASUS PRIME Z490-A LGA 1200
- Intel Core i7-10700K Comet Lake
- Team T-FORCE VULCAN Z 32GB
- Onboard ethernet via Intel® I225-V 2.5Gb Ethernet
- Onboard video via Intel UHD Graphics 630

Not working Hardware that made me sad:
- SK Hynix P31 Gold NVME m2 drive

I'm using the iGPU via the i7 and mobo, connecting my monitor via the DisplayPort connector.

<h2>Working:</h2>
Graphics, Internet, Sound, AppleID, OS Updates


Did not check:
- Sleep / Wake - I'm on the computer most of the time, so I never need to sleep it.

- HDMI port


<h2>Tools I used:</h2>
gibMacOS
https://github.com/corpnewt/gibMacOS


ProperTree
https://github.com/corpnewt/ProperTree

GenSMBIOS
https://github.com/corpnewt/GenSMBIOS

MountEFI
https://github.com/corpnewt/MountEFI

VS Code - Used as a text editor




<h1>
Steps I took, essentially starting from scratch:
</h1>
tl;dr

- make usb installer stick
- create EFI folder with SSDT and ktexts
- Create config.plist
- Install and do post install copy

1:  Create USB Installer using gibMacOS
https://manjaro.site/how-to-create-macos-big-sur-11-0-1-usb-installer-for-hackintosh/

2:  Download OpenCore RELEASE
https://github.com/acidanthera/opencorepkg/releases

3:  Follow instructions on preparing EFI
https://dortania.github.io/OpenCore-Install-Guide/installer-guide/opencore-efi.html

tl;dr - the guide above will take the EFI folder in the OpenCore Release and clean it up, as this EFI folder is needed for both the usb installer and after the installation.

4:  Get your SSDT, this is CPU specific, meaning if you are not running Comet Lake / 10th gen Intel, then you need to find the correct guide to figure out which SSDT's you need.  Guide I used for Comet Lake SSDT:
https://dortania.github.io/OpenCore-Install-Guide/config.plist/comet-lake.html#starting-point

5:  Get Ktexts:
Your Ktexts are essentially your motherboard specific drivers.  If you have different hardware, then you will need to search out which ktexts you need.

6:  Create config.plist
https://dortania.github.io/OpenCore-Install-Guide/config.plist/#amd

The config.plist is VERY important and links everything together.

7:  After we have our base config.plist, I use the Comet Lake guide, in step 4, which tells you to use ProperTree to load your SSDT's / Ktexts into your newly created config.plist.  The guide will then ask you to modify specific entries inside config.plist , which I followed.

NOTE:  I have a visual disability, and ProperTree by default has a white background which makes it hard for me to see / read.  I was able to load the SSDT and ktexts and then save config.plist  Afterwards, I used VSCode to actually make my edits based on the guide.  Lastly, when I was done making my edits in VSCode, I opened up config.plist one last time in ProperTree to make sure it CAN open the file without errors.  This is because I originally used nano, the terminal text editor, to make my changes, but nano actually screwed up the config.plist formatting which gave me errors when I was installing OSX.

8:  Use GenSMBIOS to add information to your config.plist , more information on how to use it is in the guide from step 4.
NOTE:  GenSMBIOS is important because it allows you to use AppleID in OSX as well as properly tailor your CPU information (iMac 2,1) to your hackintosh.

9:  Use MountEFI to mount the secret partition in your USB.  This secret partition, aka the EFI partition, was created when you create the USB installer in step 1.

10:  Copy the EFI folder that you've been working on into the secret parition of the USB drive:
cp -r <EFI FOLDER> <Secret Partition Mount Point>

aka

cp -r ./EFI /Volumes/EFI/

Note:  I'm doing everything on an existing Mac.

11:  Plug the USB your new computer, turn it on and go into the BIOS

12:  Make the changes in BIOS, per https://github.com/yilmazca/intel-i9-10900K-Asus-prime-Z490A-hackintosh
Bios Settings

    Disable
        Fast Boot
        CSM
        Thunderbolt
        Intel SGX
        CFG Lock (This motherboard directly closed "Bios version 0403")
    Enable
        VT-x
        4G Decoding
        Hyper-Threading
        XHCI Hand-off
        Os Type: Windows (You must remove all secure keys from Secure boot menu)
            if not working this, please select "Other Os"
        IMPORTANT: You must set Onboard GPU Memory to 64MB for use the graphics card without any problems Installation
IMPORTANT NOTE:  THE iGPU is specifically called: DVMT Pre-Allocated
Here's how to get to the IGPU setting:
Advanced Mode -> Advanced Tab

Sysrem Agwnt (SA) Configuration -> Graphics Configuration

DVMT Pre-Allocated -> 64M




13:  Save BIOS changes and go back into BIOS

14:  Hit F8 / Boot Menu and select the UEFI installer from your USB stick

15:  You will see a menu from the usb installer, something like:
    1:  Install Big Sur
    2:  Reset NVRAM
etc

By Default, in 5 seconds it'll go straight to #1 and begin the Big Sur installation

16:  Follow the OSX Installer instructions, ie format your HD, set language, location / time, create username, etc.

17:  After the installation is done and you are now in Big Sur, there is one more important thing.  If you eject the USB and restart your computer, you will find that you CANNOT boot into Big Sur, even though you installed it.  This is because installing Big Sur did not install any EFI / boot parameterss.
To fix this, we will first download MountEFI again.

18:  Make sure the USB is still plugged into your computer.

19:  Use MountEFI to mount the EFI partition from your USB stick.

20:  Copy the EFI folder from the USB driive into your computer.
ie:
cp -r /Volumes/EFI/EFI ~/Documents/

21:  Eject the USB stick from your computer

21:  Use MountEFI to mount your hard drive that OSX is installed in.
NOTE:  When OSX installs on your hard drive, it creates a secret boot partition, which is currently empty.  We use MountEFI to mount it to your computer so that we can accesss it.

22:  Copy the EFI folder into the secret boot partition, essentially what we did via the usb in step 10.
cp -r ~/Documents/EFI /Volumes/EFI/

23:  If you now restart your computer without the usb drive, your computer will see the UEFI boot configuration, which will allow you to boot into OSX without the USB.

24:  Done!



<h1>Troubleshooting Problems I had to figure out how to fix:</h1>

Kernal Panic during installation:
I got a fatal panic message that roughly says:
'nvme: "Fatal error occurred. CSTS=0x0'
It took awhile to realize that the message told me literally what the problem is, which is that the m2 nvme hard drive I bought, SK Hynix P31 Gold, DOESNT WORK with hackintoshes.  To fix, I had to remove this m2 drive from my computer and just install OSX on a SSD I had.

Error messaging during installation process:
"Failed to parse data field as blob with type boolean"
I figured out that this is a mistake I made when I was working on config.plist in step 7.  This is why its really important to get the config.plist correct and modifying the fields correctly and verifying ProperTree can open your config.plist without problems before you copy it over to the USB partition.

No internet during installation or afterwards with Intel® I225-V 2.5Gb Ethernet:
When installing oSX, it will check if it can use DHCP to get an ip address / connect to the internet.  I had what I thought was the correct ktexts to get me internet, specifically:
FakePCIID_Intel_I225-V.kext
FakePCIID.kext
It turns out we need to make sure we have not only these ktexts, but that our config.plist references our motherboard devices properly.
In config.plist there is a section "DeviceProperties".
Inside that section, in the Add area, we are suppose to copy over our motherboard device details.  From what I can tell, the basic device is ethernet, sound and something else that i'm not too sure of.  But specifically, if we don't have an entry for
PciRoot(0x0)/Pci(0x1C,0x4)/Pci(0x0,0x0)
We don't have internet, even if we have the ktexts.









