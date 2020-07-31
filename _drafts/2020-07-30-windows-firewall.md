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

For a while i've been wanting to properly configure the firewalls on our Windows servers and workstations to only allow the traffic that was really needed.  Part of the reason for taking so long is that the Windows firewall can be a bit daunting and difficult to understand, and also I didn't want to break any devices or services that were in production.

Here i'm going to write about how I went about properly configuring the firewall on our devices without breaking anything in the process and also what I learned while doing it.

## Goals

What do we want to gain by properly managing the firewall on our hosts?  The main advantage for us is to limit the exposure of our workstations and servers to help reduce or slow down malicious users or network worms.

This is usually obvious with servers, but a lot of the time the firewall on workstations is neglected or even turned off.  There is often a lot of trust placed in the local network.  Unfortunately we need to start taking the zero-trust approach, and carefully manage what users and devices have access to. 
You might think that a border firewall is good enough, and they are definitely important, but what happens when someone or something gets into the network? If we use the host firewall to stop hosts from being able to communicate with each other, we will make it much harder for a malicious person or worm to move around the network.


## Best Practice

The best place to start is to watch [this presentation](https://channel9.msdn.com/Events/Ignite/New-Zealand-2016/M377) by Jessica Payne at Microsoft.
<iframe src="https://channel9.msdn.com/Events/Ignite/New-Zealand-2016/M377/player?format=html5" width="640" height="360" allowFullScreen frameBorder="0" title="Demystifying the Windows Firewall – Learn how to irritate attackers without crippling your network - Microsoft Channel 9 Video"></iframe>

This video gives a great overview of how we should be configuring the firewall on our devices. 

Useful points:
- Use group policy objects to manage the firewall
- Only open ports and services that are required
- Don't restrict outbound traffic unless there is a very good reason.

### Use group policy objects

By using group policy objects, we can centralise the configuration of our firewall rules.  It also allows us to use organisational units to layer firewall rules and most importantly, it allows us to disable local rules and only apply the rules we specify. Disabling local rules is important as the default firewall configuration in Windows is very open and a lot of software pokes holes in the firewall when it is not required.

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

At the Workstations level, we can apply general rules that should be on all workstations, for example, rules that allow  management systems to communicate with the hosts as well as remote access tools used by systems administrators and desktop support staff.  

From there we can apply more specific rules to lab and staff workstations and further down the tree.  That way we aren't opening a port on a graphic designers computer that is only required by the developers.  This leads us on to the next point.

### Only open ports that are required

We need to think about what is really required to be open on the firewall.  Even though a piece of software opens a port when it is installed doesn't mean that it needs that port open to function.  This may take some planning and testing to get right.

I worked this out in our environment by looking at the firewall logs and identifying what traffic was actually flowing.  I then assesed whether that traffic was required before creating a rule for it.

### Don't restrict outbound traffic

Obviously there may be a very good reason to do this, but in the majority of cases it isn't worthwhile and can requires a lot more management.  If you want to restrict outbound traffic, at least start with inbound first.  Don't try to attempt to do it all at once.

Watch Jessica Payne's presentation above for more reasons why it isn't a great idea.

## Where to start?

So you are ready to tackle this, but where should you start?  

## Connection security rules

## Logging

## Testing

