---
layout: post
title: Start a web server and share files in two steps
---

Today I ran into a situation where I think Docker can help.

My colleague was working on the front end of a web application, and occasionally he needs me to make a quick fix on a REST service. I would make the change, build the war file and  scp him the war file. He need to enter his password on my desktop for the scp. 

After this happened several times, I realized that we need a better way to share the file.

Before, I would just install a web server on my Linux desktop and create a directory there and link the file in that directory, but with docker I did these:

    sudo docker pull fedora/apache
    sudo docker run -d -p 0.0.0.0:6789:80 -v `pwd`:/var/www/html/share fedora/apache

Now I have a web sever listening at port 6789, and my colleague could access the link `http://myhostname:6789/share` for all the files in this directory.

This is quite convenient and secure. No configuration file change is needed.

Some highlights about docker:

* It downloads the images and its dependency from docker hub automatically, just like using `apt-get`  to install a package
* It launches a lightweight VM(Container)
* It can map the port of the container to a port of the host(-p 0.0.0.0:6789:80)
* You can mount a directory from the host to the container(-v \`pwd\`:/var/www/html/share). This is much neater than creating an soft link.


 
