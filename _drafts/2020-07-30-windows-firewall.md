---
#layout: posts
title: Lessons learned while locking down the windows firewall
categories: [Windows, Security]
header:
  image: /images/windows-firewall/mainimage.jpg
  caption: "Photo credit: [**Markus Spiske**](https://unsplash.com/@markusspiske)"
toc: true
toc_sticky: true
excerpt: I've been working on restricting the windows firewall on servers and workstations to only the required services.  This post will covers some lessons I have learned and useful information I have found.
---

For a while i've been wanting to properly configure the firewalls on our Windows servers and workstations to only allow the traffic that was really needed.  Part of the reason for taking so long is that the Windows firewall can be a bit daunting and difficult to understand, and there isn't a lot of information available.  I also didn't want to break any devices or services that were in production.

Here i'm going to write about how I went about properly configuring the windows firewall on our devices without breaking anything in the process, and also what I learned while doing it.

# Goals

What did we want to gain by properly managing the firewall on our hosts?  The main advantage is to limit the exposure of our workstations and servers to help reduce or slow down malicious users or network worms.  

This is usually obvious with servers, but a lot of the time the firewall on workstations is neglected or even turned off.  There is often a lot of trust placed in the local network.  Unfortunately we need to start taking the zero-trust approach, and carefully manage what users and devices have access to. 
You might think that a border firewall is good enough, and they are definitely important, but what happens when someone or something gets into the network? If we use the host firewall to stop hosts from being able to communicate with each other, we will make it much harder for a malicious person or worm to move around the network.


# Best Practice

The best place to start is to watch [this presentation](https://channel9.msdn.com/Events/Ignite/New-Zealand-2016/M377) by Jessica Payne at Microsoft.
<iframe src="https://channel9.msdn.com/Events/Ignite/New-Zealand-2016/M377/player?format=html5" width="640" height="360" allowFullScreen frameBorder="0" title="Demystifying the Windows Firewall – Learn how to irritate attackers without crippling your network - Microsoft Channel 9 Video"></iframe>

This video gives a great overview of how we should be configuring the firewall on our devices. 

Useful points:
- Use group policy objects to manage the firewall
- Only open ports and services that are required
- Don't restrict outbound traffic unless there is a very good reason.

## Use group policy objects

By using group policy objects, we can centralise the configuration of our firewall rules.  It also allows us to use organisational units to layer firewall rules and most importantly, it allows us to disable local rules and only apply the rules we specify. Disabling local rules is important as the default firewall configuration in Windows is very open and a lot of software pokes holes in the firewall even though it isn't required.

Layering firewall rules also makes things a lot more manageable.  Take this orginisational unit structure as an example:

```
↳ Workstations
  ↳ Lab Workstations
    ↳ General Purpose
    ↳ Training
      ↳ Cisco Training
      ↳ Microsoft Training
  ↳ Staff Workstations
    ↳ Developers
    ↳ General
    ↳ Graphic Designers
```

At the Workstations level, we can apply general rules that should be on all workstations, for example, rules that allow management systems to communicate with the hosts as well as remote access tools used by systems administrators and desktop support staff.  

From there we can apply more specific rules to lab and staff workstations and further down the tree.  That way we aren't opening a port on a graphic designers computer that is only required by the developers.  This leads us on to the next point.

## Only open ports that are required

We need to think about what is really required to be open on the firewall.  Even though a piece of software opens a port when it is installed doesn't mean that it needs that port open to function.  This may take some planning and testing to get right.

I worked this out in our environment by looking at the firewall logs and identifying what traffic was actually flowing.  I then assesed whether that traffic was required before creating a rule for it.

## Don't restrict outbound traffic

Obviously there may be a very good reason to do this, but in the majority of cases it isn't worthwhile and can requires a lot more management.  If you want to restrict outbound traffic, at least start with inbound first.  Don't try to attempt to do it all at once.

Watch Jessica Payne's presentation above for more reasons why it isn't a great idea.

# Implementation plan

Here is the process I went through when implementing group policy based firewall control.

The main thing to remember with the implementation is that you can work piece by piece, area by area and test along the way. You don't need to have everything done in one go.  I started with the servers first, and worked on a single server/service at a time.

## Identifying services and ports

The first step was to identify services that are needed on our network.  

I started with the tools that we use to manage servers and workstations:

- Remote Desktop
- File and printer sharing
- Monitoring services
- etc

I found these the easiest and they generally apply to all workstations or servers.  It was also useful to check the documentation for the software we are running to see which ports they recommend you have open on the firewall.

To make sure that nothing was missed, I configured the firewall logging on our servers and workstations to log successful packets and increase the log file size.  I could then look through the logs to see what traffic was currently being allowed in and identify the legitimate and required traffic.

To make understanding the firewall logs a little easier, I created some PowerShell scripts to process the log files and summarise the data, which I cross-checked with the documentation as well as sites like the [speedguide.net ports database](https://www.speedguide.net/ports.php).


## Creating rules

Using the data I gathered I could can start creating rules.

## Testing and experimenting

For some critical services I wanted to be able to test my new firewall rules before deploying them into production.  For these cases, I would create a temporary virtual machine in a test OU where I could apply the rules and validate that everything worked as expected.  The same approach was used for workstation firewall rules as well. 

For non-critical services (where users wouldn't notice a small outage) I was able to apply the rules to the production server, validate that they worked as expected and if not, quickly roll back the change.  

# Terminology and concepts

Let's cover some terminology and concepts that apply to the Windows firewall.

## Profiles

The Windows firewall has what are called profiles.  These allow the firewall to apply different rules depending on the type of network you are connected to.

There are three profiles:
- Domain - For when the computer can authenticate to a domain controller
- Private - For when you are at home or on a trusted network
- Public - For when you are connected to a public or untrusted network

For servers you can generally leave rules assigned to all profiles.  For workstations it's worth thinking about when the rules might be needed, for example, you probably don't need RDP and SMB open when the user is connected to a public network.









<!-- ## Connection security rules -->