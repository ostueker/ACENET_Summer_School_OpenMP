---
layout: page
title: Setup
permalink: /setup/
---

While you can use any appropriately configured Linux system to follow along with this course, we will be assuming that you have Compute Canada account. You can log into either of the general-purpose clusters, namely

- graham.computecanada.ca
- cedar.computecanada.ca

If you are running either Linux or Mac OSX, you can do this by opening a terminal window and logging in with 'ssh username@cluster.computecanada.ca'. If you are running Windows, then you will need to use either putty or Mobaxterm.

Once you are logged in, you can request an interactive session on a compute node with the command

~~~
salloc --time=3:00:00 --cpus-per-task=4 
~~~
{: .bash}

