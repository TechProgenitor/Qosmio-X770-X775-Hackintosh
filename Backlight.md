# Enabling Keyboard Backlight on macOS and Linux with Custom ACPI Patches

Getting the keyboard backlight to work on macOS was nearly an impossible feat until a fortuitous mishap with Microsoft’s automatic Windows updates led to a breakthrough. During an update, I forced my laptop to shut down and then booted into macOS, only to discover that the keyboard backlight was functioning perfectly.

## The Discovery

Through some experimentation, I found that the keyboard backlight activates in macOS when you shut down the laptop in Windows and then hold down the power button to force the laptop off. However, if you shut down the laptop and unplug the battery, the backlight does not come on in macOS.

This led me to hypothesize that the issue might be related to the laptop’s embedded controller (EC). Further testing revealed that rebooting into Windows and then back into macOS would turn off the keyboard backlight again.

## Investigating the Embedded Controller

Using [RU.exe](http://ruexe.blogspot.com/), I compared the embedded controller’s state before and after forcing the laptop off. I noticed a consistent change in an offset value. This prompted me to experiment with writing the changed value into the EC, which successfully turned the keyboard backlight on in macOS!

Backlight Off  |  Backlight On  |  Changed Offset
:--:|:--:|:--:
![](https://github.com/user-attachments/assets/87d72b9d-7030-45d0-b08d-8f0583060d7c)  |  ![](https://github.com/user-attachments/assets/71067505-c4e0-4dd0-996d-c6e963962bb2)  |  ![](https://github.com/user-attachments/assets/50d23247-f9c1-48ca-8670-6b6dbb4edfb0)

## Crafting the ACPI Patch

Based on this discovery, I decided to create an ACPI patch to force the backlight on. By examining the field declarations of the EC’s I/O space, I found a variable named `ILED`. An ACPI debugger revealed that this variable’s value changes from 0 to 1 when the offset is modified with RU. Consequently, I wrote an SSDT that sets this variable to 1, which enabled the keyboard backlight.

![](https://github.com/user-attachments/assets/24fc9c75-c10f-4355-a7f4-250325815ef6)

## Additional Enhancements

While working on this, I also noticed that the eco mode button's green light wouldn’t illuminate in macOS. I discovered an EC variable named `ECOB` (presumably for the eco button). Writing a value of 1 to this variable successfully turned the green light back on. I created an SSDT to control both the keyboard backlight and the eco button based on the context.

Eco Mode Off  |  Eco Mode On  |  Changed Offset
:--:|:--:|:--:
![](https://github.com/user-attachments/assets/26b37a05-3108-478d-ab28-2b25e04f639c)  |  ![](https://github.com/user-attachments/assets/0090d7ef-6431-407f-9275-d4bd3f888295)  |  ![](https://github.com/user-attachments/assets/ccfc6035-14a7-475f-a73f-e6fb3d813805)

## Linux Compatibility

The same keyboard backlight issue exists on Linux distros for this laptop. Fortunately, since OpenCore injects SSDTs into all operating systems, this custom SSDT also resolves the issue on Linux.

![](https://github.com/user-attachments/assets/ccea4b81-0f01-45a3-823e-cce86cffbb36)