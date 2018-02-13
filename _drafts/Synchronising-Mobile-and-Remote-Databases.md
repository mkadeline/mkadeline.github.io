---
layout: post
title: Synchronising Mobile and Remote Databases
description: Synchronising mobile and remote databases without the use of mobile first options
date: 2017-12-06
category: datanet.jpg
excerpt_separator: <!--more-->
image: /assets/images/cardimages/
type: BlogPosting
---

With the rise of tools such as Google's firebase, the Realm suite or CouchDB and others, synchronising data between NoSQL databases is incredibly easy. Mobile databases can be updated offline and the changes filtered to remote backups and other mobiles when back online. As far as I can see, there are few options for similar behaviour with relational databases. 

I was recently tasked with a project that required databases to be replicated across mobiles and a remote source. The data model was deeply relational. Each object was essentially a collection of other objects, similar to linked tree structures, where many objects could sit in many trees. To improve analytics and to scale the trees efficiently I wanted to use a relational model. I also wanted the web interface to be much more feature rich than the mobile offering, which meant the mobile database would actually be a subset of the remote database, which was connected to the web interface. The mobile interface would be built as a limited version, with the ability to increase its ability later.

This meant I needed to efficiently reflect relevant chnages to the remote database on mobile and vice versa. 
