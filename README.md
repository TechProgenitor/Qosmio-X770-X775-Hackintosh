<img src="https://github.com/user-attachments/assets/34ec8fe4-999e-44d2-b940-0c5df1404cf1" width="497" height="349">

[![](https://img.shields.io/badge/Bootloader-OpenCore_1.0.1_RELEASE-blue)](https://github.com/acidanthera/OpenCorePkg/releases/tag/1.0.1) [![](https://img.shields.io/badge/macOS-Sierra%2010.12.6-red)](http://updates-http.cdn-apple.com/2019/cert/061-39476-20191023-48f365f4-0015-4c41-9f44-39d3d2aca067/InstallOS.dmg) [![](https://img.shields.io/badge/Qosmio-X775--3DV78-brown)](https://support.dynabook.com/support/staticContentDetail?contentId=3044152&isFromTOCLink=false)

# Please Note

Make sure "USB Legacy Emulation" is enabled in the BIOS before installation, and connect your bootable flashdrive to one of the three USB 2.0 ports. The flashdrive won't boot off of the one USB 3.0 port.

Upgrading the internal WiFi card to an [AzureWave AW-CE123H](https://www.amazon.com/AzureWave-AW-CE123H-Bluetooth-Half-Size-PCI-Express/dp/B00HRFS1GQ/ref=sr_1_1?crid=1V4SI3W1ZT3TN&dib=eyJ2IjoiMSJ9.kU9v0_7rDG_RjxkF2PT_Kw.g0iksApqUZcNHRfSeH9XrXjre1BcMhF4VsjqEZaheAk&dib_tag=se&keywords=azurewave+ce123h&qid=1726244433&s=electronics&sprefix=azurewave+ce123h%2Celectronics%2C132&sr=1-1) is strongly recommended. The Atheros AR9285/AR3011 doesn't support 5GHz networking, and bluetooth injector kexts aren't reliable. During the upgrade, you'll need to [mask pin 51 of your card](https://www.insanelymac.com/forum/topic/348335-broadcom-20702a3-bluetooth-problems/#comment-2763643) before installation to bypass Toshiba's bluetooth whitelist. [This is the tape that I personally used to mask my own card.](https://www.scotchbrand.com/3M/en_US/p/d/cbgnhw011172/) You can cut it with scissors to size, and apply it directly over the pin. 

Enable only one of the following SSDTs within the included config.plist depending on your installed card.
* SSDT-AR9285.aml allows WiFi on the stock card and is enabled by default.
* SSDT-BCM4352.aml allows WiFi on the above recommended card and is disabled by default.

DummyPowerManagement has been enabled by default in order to reach the macOS installer. You can disable this quirk after installation once you've generated a suitable SSDT-PM.aml for your CPU model.
* This repo already has an SSDT-PM.aml that's designed for the Intel Core i7-2860QM.
* If you don't have the same CPU model, you'll need to generate a new SSDT-PM.aml with [ssdtPRGen.sh](https://github.com/Piker-Alpha/ssdtPRGen.sh) and replace the existing file within the ACPI folder.

USB 3.0 doesn't work right out of the box. This can be fixed by installing a modified version of AppleUSBXHCI.kext from macOS Yosemite. Until the binary patches for this kext are discovered for macOS Sierra, the following steps can be used to enable the port.
1. [Create an account on InsanelyMac](https://www.insanelymac.com/forum/register/) if you don't already have one.
2. Download AppleUSBXHCI v710.4.11 (10.10.5).zip from [here](https://www.insanelymac.com/forum/files/file/150-patched-appleusbxhci/)
3. Open Finder and go to `/System/Library/Extensions/IOUSBHostFamily.kext/Contents/PlugIns`
4. Rename AppleUSBXHCI.kext to AppleUSBXHCI.kext.bak
5. Open the downloaded zip file, and place the extracted AppleUSBXHCI.kext in the same directory.
6. [Download Kext Utility](https://cvad-mac.narod.ru/files/Kext_Utility.app.v2.6.6.zip) and run it to automatically repair permissions and update the system cache files.
7. Restart the computer, and the port should become functional.

Sound won't work by default after installation. To fix this, go into System Preferences, select Sound, and on the output tab select "Internal Speakers". This only has to be done once.
* The audio codec in this laptop separates the subwoofer and tweeters onto two different pin complex nodes. 
* To group them, an additional audio output (named “Digital Out”) routes the subwoofer signal to the "Internal Speakers" (tweeters).

# Limitations

This repo is currently designed for Qosmio X770-X775 laptops that have the internal display and external video outputs directly connected to the discrete graphics card. (Integrated graphics don't show up and aren't available to the user.)

Due to this, the repo is only designed to work on macOS Sierra (without NVIDIA Web Drivers or CUDA installed). The Fermi graphics card that the laptop comes with causes issues in other macOS versions.
* In macOS versions below Sierra, the notorious "Fermi freeze" problem occurs requiring the user to manually turn off and on the computer again.
* In macOS versions above Sierra, hardware acceleration is very glitchy. Disabling hardware acceleration fixes this (add nv_disable=1 to boot-args), but creates a sluggish experience for the user.

Thankfully, the discrete graphics card comes in the MXM 3.0b format. It should be possible to upgrade this GPU to something that isn’t so problematic. If anyone's able to upgrade the card successfully, please open an issue.

If you have a version of this laptop with NVIDIA Optimus, the repo as it is currently won't work for you. This is because you'll need to instead use integrated graphics which aren't supported by the repo at this time. You'll likely need to do the following in order to add support:
* Modify this repo's config.plist by patching the iGPU framebuffer.
* Disable the NVIDIA GPU with an SSDT.
* Enable brightness control by modifying SSDT-PNLF.aml and/or adding device properties to the iGPU.

If you’d like me to create a modified EFI for the integrated graphics, or if you have the capability to add support, please don’t hesitate to open an issue. I have an idea of how to modify this repo's EFI, but I’m unable to test the modifications for accuracy and stability, and I’d prefer not to make any assumptions.

# ⚠️ Warning

The Qosmio X770-X775 laptops experience random shutdowns due to problems with both the power button and the BIOS. In order to have a pleasant Hackintoshing experience, you might need to replace your power button and/or [downgrade your BIOS to the initial production version.](sby5v110.exe)

* The power button is a cheaply designed membrane with a paper thin ribbon cable. The membrane suffers from a defect where it’ll eventually stick down when powering on the laptop resulting in a shutdown. I’m assuming that Toshiba eventually fixed the part, as I haven't encountered this problem. If you need a replacement power button, you should be able to purchase one online that doesn’t have the defect. This also affects similar Toshiba Satellite series variants. Alternatively, you can disconnect your power button, and simply tap the play/pause button on the touchbar to turn on this laptop.

* There’s buggy GPU thermal throttling code in BIOS updates beyond the initial production version. Instead of throttling the card when the EC issues a VGA protection function, the laptop will just force itself off. Toshiba (now DynaBook), doesn't host the initial production BIOS file (Version 1.10) that's free from this problem. It’s been included here in this repo for those who'd like to downgrade their BIOS.

Don't disconnect the primary hard drive ribbon cable from the laptop's motherboard. For me, it was nearly impossible to reconnect that cable, and I ended up tearing off the blue plastic tab that's attached on top of it. Due to this, I couldn't reconnect the cable back into it's connector because it wasn't thick enough to stay in place. In order to solve this, I had to superglue the plastic tab back onto the ribbon cable, and eventually succeed in shoving the miserable thing back into its connector without the tab sliding off.

# Specifications

| Hardware | Qosmio X775-3DV78 |
| ------------- | ------------- |
| CPU | Intel Core i7-2860QM |
| GPU | NVIDIA GeForce GTX 560M |
| Display | Samsung SEC5044-173HT02-T01 |
| Chipset | Intel HM65 |
| Memory | 2x - Corsair CMSO8GX3M1A1333C9 |
| Wi-Fi | AzureWave AW-CE123H |
| LAN | Realtek RTL8111 |
| Audio | Realtek ALC269VB |
| Trackpad | Synaptics TouchPad V7.4 on PS/2 |
| Card Reader | JMicron JMB38X |

# Issues
| | |
| --- | --- |
| SD card slot kexts make macOS unstable.| JMB38X.kext and HSSDBlockStorage.kext have been disabled by default within the config.plist. A new kext might need to be written for the card reader. |
| Trackpad gestures are sluggish. | Unsure if this can be fixed. You'll also notice that accessing the trackpad settings within "System Preferences" results in a crash. This is caused by some sort of macOS bug related to the Fermi GPU framebuffer. Disabling GPU acceleration (via the nv_disable=1 boot-arg) seems to fix it.
| Disable ringFreqTables kernel patch doesn't work above macOS Sierra or on macOS installers. | Either need to tweak the config.plist file or write a Lilu plugin. |
| Boot picker screen resolution is less than 1080p. | Requires modifying the VESA tables of the GPU, but unsure where the VBIOS is stored or how to flash it. Upgrading the GPU entirely would also likely resolve this issue and is a more preferred solution. |
| Boot picker cursor responsiveness is choppy. | I’m not sure what’s causing this issue. If you want to hide the cursor, you can do so by opening the config.plist and setting the "PickerAttributes" value to 0. |