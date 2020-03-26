title: Use Both Docker and VMware on Windows 10
author: John Miroki
date: 2018-09-18 12:27:08
tags:
---
It's actually impossible to do so. The two softwares require the opposite of sysmtem configuration. On my Windows 10, I need to do two things to switch the envirenment so that either of them can work.

1. Hyper-V  https://blogs.technet.microsoft.com/gmarchetti/2008/12/07/turning-hyper-v-on-and-off/

2. Memoery Isolation http://www.win10.today/jc/7941.html

Docker needs both features **on** while VMware needs both of them **off**.