---
layout: post
title:  "6 Months of honeypot traffic analysed"
date:   2020-1-5 00:01:00 +0000
categories: Talk
---

# 6 months of honeypot traffic analysed.
A while back for a minor project i setup a cheap vps to run it as a honeypot to track the types of attacks and traffic going around the internet constantly. These findings were meant to be included in a conference talk but i figured i would write something about it regardless.

# Setup
If you wanted to replicate my setup heres how. Keep in mind that my misconfigurations did cause me issues and time, so the tips are to help.
**- Rent a VPS**
 
I went with the lowest spec box on DigitalOcean as it was the cheapest, however if you can afford to, i would recommend getting a slightly beefier box for reasons which will be clear soon.
**- Install a unix os**

Not too different which os you chose to go with, ubuntu server was my choice because it works and had everything i needed for this. 
**- Install a honeypot.**

I chose to use 'cowrie' as it had a well maintained repo and good documentation on setup. It also uses python virtual environments so the chances of a breakout is low.
**- Run a splunk or ELK instance.**

Heres where i messed up. I should have got a higher spec'ed box so i could have run an ELK stack locally for ease of life. Instead i ran a local splunk vm. Dont do this. Cowrie, and many other honeypots have forwarding support, meaning they can send the logs they make direclty to a splunk/ELK instance. If you have one of these running on the vps at the same time, all your logs get aggregated automatically and you can live peacefully. Because i ran a splunk box locally, i had to send the logs over ssh to my vm and manually import into splunk. Not fun.

**- Do some hardening**
Just a tip, because your box will be internet facing (and most likely in a likely ip range), its a good idea to do _some_ basic hardening. As we will see, vulnerable hosts often become slaves to running malware hosts and C2s, dont let this be you.
Also make sure to excersise caution, dont run anything you find in the honeypot logs as root on the box for example. Make sure to change the ssh config so you can log back in and maybe do some config with iptables just incase.

Overall for ~6 months of hosting, this project cost me about $50 AUD, but there is return on investment.

# Splunk? ELK?
Having a traffic corelation / visualisation platform like splunk or Elasic search will be perpetually better than reading json logs. It also allows you to make graphs and sort throught the data easily. Both of these platforms run a sql like language that can easily filter out specific demands. (eg. source ips or input data)
If youre struggling to chose a setup i would recommend splunk as the language is very simple and setup with forwarding is a breeze.
# External tooling
If you wish to look into reversing or monitoring malware and file samples you get presented with, its a good idea to either;
**- Run a sandbox vm**
	- Microsoft offers free iso files
	- run wireshark and or file system monitoring tools
	- Make sure the environment is disposable
	- Use snapshots
**- Use online solutions**
	- Hybrid analysis
	- virus total
	- Any.run

# Findings

# Main types of files
~> Cryptocurrency miners.
These hit almost immediatly, a sample of a chinese monero miner was detected within just hours of the honeypot going live.
~> Botnet recruiters
These showed up gradually but there seemed to be a constant flow. Their purpose is to act almost like persistence on the host then to act on recieved botnet commands from their host.
~> Router malware
This type there was less of. The majority of the ones i found i actually ended up pulling from open ftp servers because the dumped command had the name wrong. 'wget -r' served a great use here.
~> skid tools
Every once in a while i would find a sample of malware that contained strings that i would consider to be linked to skid tools. I managed to pull a specific file off a server due to the bot requesting it on the honeypot. Once i had the file in my vm i took it apart with ghidra so i could try to understand what it was doing. Turns out i didnt need to do this. With some google dorks and the unix 'strings' application i was promptly led to a public pastebin post of the source code leaked off of a forum.
This is i guess classed as a botnet recruiter but the humor of it led me to class it differently. Most of the samples i got were from the same source but slightly tweaked, all advertising a telnet scanner, bot killer and udp/tcp flooding.

# Noticable events
- First malware sample

As mentioned above, this event happened just hours after setup. A monero miner sample got dropped into the honeypot and would have begun mining if it were a normal system.
- Indexed by russian proxy site

This also happened fairly quickly, and if im honest something i didnt see happening at all. It caused my boxes ip to be put on a proxy list which seemed to amplify the traffic through http requests.
- First manual user interaction after automated attempt at same IP address
Yes, im not making this up. Someone legitimately logged into the honeypot manually trying to debug their downloader script for their malware. I just happened to be watching the logs at the time when i noticed someone login and execute portions of a previously executed code snippet. When the user had tried to debug their code, they dropped a 'rm -rf /' before exiting. 



# Proxy traffic

Main types of sites requested through proxy
- Yandex.ru
- Twitch.tv -> viewbotting -> phishing campaigns
- Shopping sites -> referal abuse
- Match.com -> potential romance scams
- IP checking sites -> some of which are C2s
- Pr0n sites -> ¯\\_(ツ)_/¯
- C2s / malware hosts
- Dead domains -> echos from dead malware
- idk -> i cant explain some hosts

Why did this happen?
Well in a normal scenario with open or accessable ssh, a normal user would in fact be able to proxy traffic through the host. However in this case, the honeypot disallows any of these actions. The requests still come through but are never acted upon.

# Geolocation
```
Top 3 countries by interactions
1. Ireland 
2. Germany 
3. Russia 

``` 

wtf ireland? not china/russia?
I asked myself this exact question when splunk spat these numbers out, i didnt trust it so i re-ran the queries through splunk again just to check. Reasons for this output is not concrete in my findings but something that may explain it is the fact that Ireland is a tax haven for large corporations (amazon, google etc.) and they have large data centres there that is causing the traffic. That or there is a massive proxy operation happening.


# General Stats
```
Malicious file download attempts =	 ~4500 
Unique source ips =			 ~9k
Unique destination ips =		 ~4k
Top interactions from single ip = 	 >2m
Attempts at /etc/passwd =		 >1500
Attempts at /.ssh/authorized_keys =	 >1200
Unique usernames =			 ~6k
Unique passwords =			 ~45k
Total events* =				 ~20m

20m/6 months = 110k per day

* not counting scans. Attempt or input only.
```

# Returns
- 45k wordlist created from logins and attempts
- malware samples to disassemble and reverse
- hours of fun


# Take aways
- There is loads of malicious traffic that hits endpoints all the time. 
- People are using bots for all kinds of things but mainly for profit
- Unprotected servers play host to free C2/malware drop hosting
- People try all kinds of things when given free access
- Some people havent grasped how to transfer files on unix systems.

# Things to do differently
As i mentioned i would certainly take advantage of forwarding from the honeypot to a running ingestion log service like logstash. I would also setup a visualisation panel like greylog or tango for splunk. Just for general interactions i probably would have chosen a host with better latency, i seemed to skim over that option on setup :facepalm:. I would also develop a better warning system for breakouts, i know the python virtual env structure is amazing, but incase of some new exploit developments i would want to know who or what is on my system. At the time, the only warning system i had was emails from the vps provider letting me know theyre terminating my service due to illicit activity.

# Further research
Reflecting on this project, i got thinking about what i could do to further gather information on current botnets and malware floating around the internet. If you havent seen the defcon 20 talk 'owning bad guys and mafia with javascript botnets', i recommend sitting down and watching it. I rewatched it whilst sifting through logs one night and began to wonder if i could replicate the same thing but with ssh. What if we could activily allow connections through an ssh tunnel on a honeypot box and strip out credentials and addresses to better see what bots and adversaries are up to.
Ive started looking into the difficulties of doing so and will post the project to github if anything comes of it.
