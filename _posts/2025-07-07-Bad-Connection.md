---
layout: post
title: Bad Connection
tags: network
---

I was trying to push a post to my github repo of my github blog. But, it kept not happening. The 'git psuh' kept getting stuck. A network reconnect and a ping was done. Pinging github.com showed most pings timing out. Although, the domain name was resolved, the pings failed. Why does this happen?

Obviously, the connection was good enough to resolve the domain name, the DNS servers worked. But, the subsequent parts did not. My guess about the subsequent parts is that, that part must be some kind of switch, routing my requests to the next switch. My guess is that, that next part, the switch failed. And my guess is that, the monitoring of the switch is manual. Somebody is supposed to restart that switch. But, somebody did not do that. So, all my TCP connections kept going via that switch, which had failed. So, none of my requests would work, neither the 'git push', nor a browser request. It would keep on happening, until that switch is restarted. This is my guess.

How many times does this happen? I can go back and count it to some once in a week.
