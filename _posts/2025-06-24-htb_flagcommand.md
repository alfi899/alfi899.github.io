---
title: "HTB Flag Command"
date: 2025-06-24
categories: [HTB, Writeup, Challenge]
tags: [Web]
description: Very Easy Webchallange from HTB
pin: false
math: false
mermaid: true
#image:
#  path: /assets/img/posts/htb-machine-name/banner.png
#  alt: "HTB Machine Banner"
#  width: 800
#  height: 400
---

> **HTB Flag Command - Writeup**  
> Very Easy

---
## Writeup

Challenge Description
>Embark on the "Dimensional Escape Quest" where you wake up in a mysterious forest maze that's not quite of this world. Navigate singing squirrels, mischievous nymphs, and grumpy wizards in a whimsical labyrinth that may lead to otherworldly surprises. Will you conquer the enchanted maze or find yourself lost in a different dimension of magical challenges? The journey unfolds in this mystical escape!

If we view the website we see the following:

![Landingpage](/assets/img/htb/flag_command/image1.png)

As we can see this is a terminal based game. We have an List of diffenrent Options we can try. I also used Burp Suite to track all me requests. Since all the Options doesn't lead to anything useful i looked at the requests in burp.

And there was one interesting Endpoint. `/api/options`

![Burp](/assets/img/htb/flag_command/image2.png)

There we can see all the options we can use and there is also a `secret` Option. So i sended the request to the Repeater and tried that command.


Notice that this has another endpoint to send to `/api/monitor`.

![flag](/assets/img/htb/flag_command/image3.png)

And so I get the flag. `HTB{D3v3l0p3r_t00l5_4r3_b35t__t0015_wh4t_d0_y0u_Th1nk??}`