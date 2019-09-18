---
layout: post
title: "Allowing 80 port on EC2 instance"
author: "Jonghyun Ho"
categories: AWS
tags: [aws]
---

# Allowing 80 port on EC2 instance

Even if 80 port is configured to be open for Inbound, it's not able to access from outside.

It can be done by changing IP table with following command.
Packets coming through 80 port are redirected to 8080 port.
```
$ sudo iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
```

To check the updated IP table,
```
$ sudo iptables -t nat -L --line-numbers
```
