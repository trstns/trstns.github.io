---
#layout: posts
title: Expose the MDT progress window on Windows 10/11
categories: [Windows,MDT,Deployment]
header:
  overlay_image: /images/header/christian-wiediger-F8IAN0lyFJU-unsplash.jpg
  caption: "Photo credit: [**Christian Wiediger**](https://unsplash.com/@christianw)"
toc: false
toc_sticky: true
excerpt: How to disable the Windows first logon animation so that the MDT progress window is visible.
classes: wide
---

Windows displays a fullscreen message during the first logon to provide some context to the user while the profile is being built.  This is inconvenient when deploying a computer with MDT as it hides the progress window and gives the impression that the setup is running slow as the message remains on-screen for much longer than it normally would.  This is a quick modification to a task sequence to disable the first logon animation so that the MDT deployment progress is visible.

# Disabling the first logon animation

To disable the first logon animation we need to set the registry keys controlling it before the first user logs in.  We can do this using the Unattend.xml file for the task sequence.

Open the Unattend.xml for your task sequence task sequence in a text editor.  The Unattend.xml file is located in `[deploy share]\Control\[task sequence id]`.

In the _specialize_ pass under `<component name="Microsoft-Windows-Deployment" ...`, add the following _RunSynchronousCommand_ elements after the existing ones.

``` xml
<RunSynchronousCommand wcm:action="add">
    <Description>Disable First Logon Animation</Description>
    <Order>5</Order>
    <Path>reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableFirstLogonAnimation /d 0 /t REG_DWORD /f</Path>
</RunSynchronousCommand>
<RunSynchronousCommand wcm:action="add">
    <Description>Disable Desktop switch timeout</Description>
    <Order>6</Order>
    <Path>reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v DelayedDesktopSwitchTimeout /d 0 /t REG_DWORD /f</Path>
</RunSynchronousCommand>
```

Keep in mind that the `<Order>` numbers need to be unique, so you will need to update these if you have added other commands.

# Re-enabling the first logon animation

The changes in the previous step will remain after imaging has completed.  If you would like to revert them so that users are presented with the first logon animation, add the following elements to the _FirstLogonCommands_ in the OOBE pass:

``` xml
<SynchronousCommand wcm:action="add">
    <CommandLine>reg delete HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableFirstLogonAnimation /f</CommandLine>
    <Description>Enable First Logon Animation</Description>
    <Order>1</Order>
</SynchronousCommand>
<SynchronousCommand wcm:action="add">
    <CommandLine>reg delete HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v DelayedDesktopSwitchTimeout /f</CommandLine>
    <Description>Enable Desktop switch timeout</Description>
    <Order>2</Order>
</SynchronousCommand>
```
These should be set to run before Lite Touch, so make sure to change the order value for the Lite Touch command to 3.

**Remember**, these changes need to applied to each task sequence individually.  It is possible to update the template Unattend.xml (Located in `C:\Program Files\Microsoft Deployment Toolkit\Templates\Unattend_x64.xml`), but this would need to be done on any computer used to create task sequences.