<img src="https://github.com/user-attachments/assets/34ec8fe4-999e-44d2-b940-0c5df1404cf1" width="488" height="343">

[![](https://img.shields.io/badge/Bootloader-OpenCore_1.0.1_DEBUG-blue)](https://github.com/acidanthera/OpenCorePkg/releases/tag/1.0.1) [![](https://img.shields.io/badge/macOS-Sierra%2010.12.6-red)](http://updates-http.cdn-apple.com/2019/cert/061-39476-20191023-48f365f4-0015-4c41-9f44-39d3d2aca067/InstallOS.dmg) [![](https://img.shields.io/badge/Qosmio-X775--3DV78-brown)](https://support.dynabook.com/support/staticContentDetail?contentId=3044152&isFromTOCLink=false)

# Please Note

WiFi has been upgraded to the [AzureWave AW-CE123H](https://www.amazon.com/AzureWave-AW-CE123H-Bluetooth-Half-Size-PCI-Express/dp/B00HRFS1GQ/ref=sr_1_1?crid=1V4SI3W1ZT3TN&dib=eyJ2IjoiMSJ9.kU9v0_7rDG_RjxkF2PT_Kw.g0iksApqUZcNHRfSeH9XrXjre1BcMhF4VsjqEZaheAk&dib_tag=se&keywords=azurewave+ce123h&qid=1726244433&s=electronics&sprefix=azurewave+ce123h%2Celectronics%2C132&sr=1-1). If you've got a different card, the “compatible” property value within the config.plist file might need to be changed, or replaced with a different property altogether. If you'd also like to upgrade your WiFi card, you'll need to [mask the disable pin of your card](https://thecomputerperson.wordpress.com/2016/11/04/how-to-mask-off-the-wifi-power-off-pins-on-m-2-ngff-wireless-cards-the-old-mini-pci-pin-20-trick/) to bypass Toshiba's whitelist.

DummyPowerManagement has been enabled by default in order to reach the macOS installer. You can disable this quirk after installation once you've generated a suitable SSDT-PM.aml for your CPU model.
* This repo already has an SSDT-PM.aml that's designed for the Intel Core i7-2860QM.
* If you don't have the same CPU model, you'll need to generate a new SSDT-PM.aml with [ssdtPRGen.sh](https://github.com/Piker-Alpha/ssdtPRGen.sh) and replace the existing file within the ACPI folder.

USB 3.0 doesn't work right out of the box. This can be fixed by installing a modded version of [AppleUSBXHCI.kext](https://www.insanelymac.com/forum/files/file/150-patched-appleusbxhci/) from macOS Yosemite. The binary patches for this kext have yet to be discovered for macOS Sierra.

# Limitations

This repo is currently designed for Qosmio X770/X775 laptops that have the internal display and external video outputs directly connected to the discrete graphics card. (Integrated graphics don't show up and aren't available to the user.)

Due to this, the repo is only designed to work on macOS Sierra (without NVIDIA web drivers installed). The Fermi graphics card that the laptop comes with causes issues in other macOS versions.
* In macOS versions below Sierra, the notorious "Fermi freeze" problem occurs requiring the user to manually turn off and on the computer again.
* In macOS versions above Sierra, hardware acceleration is very glitchy. Disabling hardware acceleration fixes this (add nv_disable=1 to boot-args), but creates a sluggish experience for the user.

Thankfully, the discrete graphics card comes in the MXM 3.0b format. It should be possible to upgrade this GPU to something that isn’t so problematic. If anyone's able to upgrade the card successfully, please open an issue.

# ⚠️ Warning

There's a widespread random shutdown issue that affects this laptop lineup. People claim that it's due to an issue with the power membrane, but this is completely false. The root cause is poorly written GPU thermal throttling code in BIOS updates beyond the initial production version. Toshiba (now DynaBook), doesn't host the initial production BIOS file (Version 1.10) that's free from this issue. It’s been included here in this repo for those who'd like to downgrade their BIOS.

Regarding the power membrane, I've witnessed firsthand that it's prone to failure if you're not careful. It's a poorly designed part with a fragile paper thin ribbon cable that I've torn before on accident. However, breaking that cable will disconnect the wiring, not cause a short. As a result, this will either break your power button or cause you to lose the led indicator on the button. I don't know why the power membrane wasn't just integrated into the pcb and ribbon cabling for the touchbar.

Don't disconnect the primary hard drive ribbon cable from the laptop's motherboard. It was nearly impossible to reconnect that cable and I ended up tearing off the blue plastic tab that goes on top of it. Due to this, I couldn't reconnect the cable back into it's connector because it wasn't thick enough to stay in place. In order to solve this, I had to superglue the plastic tab back onto the ribbon cable, and eventually succeed in shoving the miserable thing back into its connector without the tab sliding off.

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
| Audio | Realtek ALC269 |
| Trackpad | Synaptics SMBus |
| Card Reader | JMicron JMB38X |

# Issues

* SD card slot kexts make macOS unstable. Associated kexts have been disabled by default within the config.plist. A new kext would need to be written for the card reader.
* Not sure how to group the subwoofer and the internal speaker output together. (Due to this, the subwoofer output is currently disabled.)
* Trackpad gestures are sluggish (might need to debug VoodooRMI).
* NVIDIA 3D Vision drivers aren't available for macOS.
* Disable ringFreqTables kernel patch doesn't work above macOS Sierra or on macOS installers. Either need to tweak the config.plist file or write a Lilu plugin.
* Boot picker isn't 1080p. Requires modifying the VESA tables of the GPU, but unsure where the VBIOS is stored.