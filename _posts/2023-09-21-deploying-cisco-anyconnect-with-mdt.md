---
#layout: posts
title: Fix 'Dirty Environment Found' message while installing Cisco AnyConnect with MDT
categories: [Windows,MDT,Deployment]
header:
  overlay_image: /images/header/mathew-schwartz-sb7RUrRMaC4-unsplash.jpg
  caption: "Photo credit: [**Mathew Schwartz**](https://unsplash.com/@cadop)"
toc: false
toc_sticky: true
excerpt: Cisco AnyConnect can cause a 'dirty environment found' error when deployed with MDT, here is how to fix it.
classes: wide
---

I was recently troubleshooting our MDT task sequence to try to find the cause of a 'Dirty Environment Found' message appearing while deploying computers.  

I traced it down to the Cisco AnyConnect installer, but it took a lot of googling and trial and error to find a way to fix it.

# The cause

It turns out that the 'Dirty Environment Found' message was appearing because the AnyConnect installer was detecting that Explorer wasn't running and trying to launch it, which re-ran the RunOnce commands and in turn relaunched Lite Touch.  The installer also seemed to stall for a while, so it was taking approximately 15 minutes to install instead of 15-30 seconds.

# The solution

One possible solution would be to set `HideShell=NO` so that explorer is running during deployment.  We have our MDT rules configured with `HideShell=YES` so students can't interfere with the computers while they are deploying, so this isn't an option for us.

The method that I found which fixes this was to temporarily replace the RunOne registry key with an empty one so that they don't execute while AnyConnect is installing.

I added the following lines to my AnyConnect install script:

``` powershell
# Move the RunOnce key so that AnyConnect will install sloothly
Rename-Item HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce RunOnce.bak
New-Item HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce

#########################################
# Place code to install AnyConnect here #
#########################################

# Put the RunOnce key back so we don't break MDT
Remove-Item -path HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce -force
Rename-Item HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce.bak RunOnce
```

This is unfortunately a bit of a hack, but until Cisco fix up their installer it's one i'll live with as it does speed up deployments by 15 minutes.