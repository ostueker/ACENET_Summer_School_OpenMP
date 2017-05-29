---
layout: page
title: Setup
permalink: /setup/
---

While you can use any appropriately configured Linux system to follow along with this course, we will be assuming that you have an ACENET account. You can log into any of the available clusters, namely

- fundy.ace-net.ca
- mahone.ace-net.ca
- glooscap.ace-net.ca
- placentia.ace-net.ca

If you are running either Linux or Mac OSX, you can do this by opening a terminal window and logging in with 'ssh username@cluster.ace-net.ca'. If you are running Windows, then you will need to use either putty or Mobaxterm.

Once you are logged in, you should get an interactive session on a compute node with the command

~~~
qrsh -cwd -pe openmp 4 -l h_rt=10:00:00 bash
~~~
{: .bash}


