---
layout: post
title:  "Why are almost all my DNS lookups failing?"
---

I have my own installation of [Arch Linux](https://www.archlinux.org). 
This means that I hand picked the OS components, and I configured things manually. Sometimes things go wrong and I have to figure out, usually without a specific guide, how to fix obscure problems. 

> Doing this has helped me understand a GNU/Linux distribution inside and out. One day I'll write a blog post about this. I'll write about the set up, about the experience, and what I have learned so far.

Today I want to tell you about how I could only visit some websites. Google worked, my most frequently websites worked, but I got IP address not resolved errors everywhere else!

I figured that something was very wrong with my DNS resolution. So I started poking around my system for the issue.

Checked my `/etc/resolv.conf` file. Seemed ok.

Used the program `dig` in the terminal to ping some failing hostnames. That seemed to work too.. which made me confused.

I didn't want to go through the hassle of auditing my network configuration as I thought I had that nailed down already.

A quick google search "dig works but dns does not arch linux" led me to this result, buried under some results: [DNS lookup fails only on specific sites while using Arch Linux](https://superuser.com/questions/1366094/dns-lookup-fails-only-on-specific-sites-while-using-arch-linux)

This Stack Exchange user seemed to be having the same problem as me, and `dig` was working for them too!

I read the one and only answer:
> Most likely it's a timezone or system time issue. Set your computer to the correct time and timezone and it will probably go away.

The long answer pointed out [DNSSEC](https://www.cloudflare.com/dns/dnssec/how-dnssec-works/) validation errors, which relies on the correct local time to do signature checking. This is great! My system is protected from DNS spoofing attacks and such, which `dig` doesn't show.

I looked at my time display widget on my i3 bar and yea.. my system time was way off! How did I not notice this?

## Time to fix my system clock

I head over to the awesome Arch Linux wiki and browse the [System time](https://wiki.archlinux.org/index.php/System_time) article.

I read about the Hardware clock vs System clock, and about Time synchronization.

Checked that my hardware clock was incorrect using `hwclock --show`

Sometimes my hardware clock goes out of sync because I dual boot Windows and it messes that up. I applied [that UTC time fix](https://wiki.archlinux.org/index.php/System_time#UTC_in_Windows) a while ago though.

I figured to try time synchronization with NTP and so I went to set that up.

Looking at the list of options I selected [`systemd-timesyncd`](https://wiki.archlinux.org/index.php/Systemd-timesyncd). I'm someone that likes systemd (bite me).

After enabling and starting the service with:
```
# systemctl enable --now systemd-timesyncd
```

I go ahead and configure the default Arch NTP servers in the file `/etc/systemd/timesyncd.conf`

```conf
[Time]
NTP=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org
FallbackNTP=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org
```

Then I wait to see some results... Which didn't show. So I check again and it turns out I need to run.

```
# timedatectl set-ntp true
```

Checking the status using `timedatectl status` I notice that it's stuck. Of course, the issue is, again, resolving those ntp hostnames, duh.

I manually set my system clock, knowing (more like hoping) that it would fix my DNS problems.

```
# timedatectl set-time "2019-05-30 22:01:25"
```

It complained about NTP being on so I had to undo the ntp flag with `timedatectl set-ntp false`

I try again to set the time, trying to be as accurate as possible to the timestamp.

I check my browser and my websites were resolving again!

But is time sync working? I turn `set-ntp` to true again and check with the status command. I see that it is working!

Finally, I fixed my hardware clock by synchronizing it to the system clock.

```
# hwclock --systohc
```

With all of this... I hope to never encounter this issue again with my system.
