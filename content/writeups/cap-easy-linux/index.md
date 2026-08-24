+++
title = 'Cap'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

![Cap.png](Cap.png)

Found nathan's credentials from a simple fuzz on the second tab of the web app - `/data/0` came back with a different response length to the rest.

The password works for both SSH and FTP: `Buck3tH4TF0RM3!`

Used them to get onto the box - the notes stop here, but this is the user flag.
