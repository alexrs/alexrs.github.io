---
title:  "A simple MessagePack decoder for TheThingsNetwork"
description: A simple MessagePack decoder for TheThingsNetwork.
date: 2023-04-14 12:00:00
layout: post
---

I've been working on a project for university that involves Arduino, LoRaWAN, and [TheThingsNewtork](https://www.thethingsnetwork.org/). To transmit data efficiently, we decided to use [MessagePack](https://msgpack.org/), which is supposed to be efficient and fast (disclaimer: I haven't done much research to confirm if this is the best option).

However, I wasn't able to find a JavaScript function with no external dependencies to decode the MessagePack message. So, with the help of Github Copilot, my debugging skills, and ChatGPT, I arrived at this solution:

<script src="https://gist.github.com/alexrs/3a00dd680578cb7f3a1f7eb70846ff7b.js"></script>

It's not a complete solution, but it seems to work for my use case. Hopefully, it can be useful for you too.
