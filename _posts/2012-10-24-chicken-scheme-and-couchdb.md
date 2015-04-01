---
title: Chicken Scheme and CouchDB
date: 2012-10-24
layout: post
categories:
- ChickenScheme
- CouchDB
tags: []
---

## Why?

I've recently been playing around with <a href="http://call-cc.org/" target="_blank">Chicken Scheme</a>, leaving poor Clojure all high and dry (minus some visits to the newly started <a href="http://www.meetup.com/DFW-Clojure/" target="_blank">DFW-Clojure User Group</a>). After building a service in C at work, I've caught the high-performance bug and Clojure hasn't particularly been helpful in that regard. I've also found Chicken Scheme to be very simple with a more straightforward syntax than Clojure. I'll still use Clojure, and am using it on a web-app side-project, but client side I'm really digging Chicken.

I've also been using MongoDB, but (OMG!) there's no driver currently for MongoDB in Chicken Scheme. Until I have the chops to write my own version, I decided I might as well learn a similar DB: enter <a href="http://couchdb.apache.org/" target="_blank">Couch</a>. In my naive view, it's very similar to Mongo - it's a document-oriented database with JSON (or <a href="http://bsonspec.org/" target="_blank">variation thereof</a>) as its representative data structure. Even cooler, it's already been ported to <a href="http://openbbnews.wordpress.com/2012/01/13/couchdb-playbook/" target="_blank">BlackBerry 10</a>. This gives me some really good ideas for new projects.

## The Setup

I'm currently running <a href="http://linuxmint.com/" target="_blank">Linux Mint</a>, a "fork" of Ubuntu, so this setup guide follows what was necessary for me to get up and going with Chicken Scheme and CouchDB.

### Chicken Scheme

There are two options. The easy way:

```
sudo apt-get install chicken-bin.
```

This installs everything for you with no issue, however it's an older version (as seems to be a common issue with the Ubuntu repositories). At the time of writing it's at 4.7.0. You can also install Chicken by compiling from source. Download the tarball from <a href="http://code.call-cc.org/releases/4.8.0/" target="_blank">call-cc.org</a>, untar, and follow the INSTALL instructions. Easy!

### CouchDB Egg

Chicken Scheme has an extension system which packages and distributes "<a href="http://wiki.call-cc.org/eggs" target="_blank">eggs</a>." An egg consists of Scheme sources plus some meta information such as build scripts and egg dependencies. You can install an egg using the chicken-install utility that comes with Chicken.

```
chicken-install -sudo couchdb
```

This will download the <a href="http://wiki.call-cc.org/eggref/4/couchdb" target="_blank">couchdb egg</a> along with all eggs in the dependency tree. The utility then begins compiling and installing all eggs along the way. If you don't do a lot of development, chances are likely that you will run into system dependencies that are unmet. Chicken Scheme compiles to C, and as such many eggs have dependencies on C libraries.

The one I hit with couchdb in particular was openssl. These system lib dependencies can be a little tricky to identify. Two tips:

 * See which egg is failing, then find it in the egg index and look through it's notes which /usually/ indicate dependencies
 * Read the error message and decipher from there.

For example, my first attempt at installing the couchdb egg failed due to missing openssl header files. The install utility was attempting to compile the openssl egg, which was a dependency of the http-client egg, which is a dependency of the couchdb egg.

A few searches for openssl and I satisfied the dependency with the following command:

```
sudo apt-get install libssl-dev
```

With that successfully in place, another call to install the couchdb egg succeeded.

## Playing Around
I didn't have time to compile and install a CouchDB server on my machine, so <em>to the cloud</em> I went. I signed up for an account at <a href="https://cloudant.com/" target="_blank">cloudant.com</a> which provides a free-for-small-databases CouchDB compatible database. After signing up, I imported the example database aptly named "crud" and began toying around.

Booting up csi, the Chicken Scheme interpreter, I called <code>(use couchdb)</code> and set out to play. The first hurdle: making a connection. The current documentation says to use something along the lines of <code>(define couch (make-connection "databasename" "http://localhost:123"))</code>.

That failed miserably.

No matter what I tried, the command <code>(get-server-info couch)</code> would fail with a connection refused. <code>(connection-uri couch)</code> showed that the URI struct was still pointing at localhost. Luckily, the <a href="http://code.call-cc.org/svn/chicken-eggs/release/4/" target="_blank">source for Chicken eggs</a> are available, even for CouchDB. The make-connection function is created by the <code>(defstruct connection ...)</code> expression. I noticed that the server attribute of connection struct is a <code>uri-reference</code>, <em>not</em> a string. Reading through the defstruct docs I figured out the following worked - woot.

```
(import uri-common)
(define cloudant-url server: (uri-reference "https://username:password@username.cloudant.com"))
(define cloudant (make-connection server: cloudant-url database: "crud"))
(get-server-info cloudant)
```

This returns JSON data in Scheme structure:

```
#(("couchdb" . "Welcome")
  ("version" . "1.0.2")
  ("cloudant_build" . "768"))
```

Since I'm new to Scheme, getting the data out was a bit odd, and I'm sure completely inefficient.

```
(define server-info (vector->list (get-server-info cloudant)))
(alist-ref "version" server-info equal?)
```

That about sums it up. Hopefully I figure more stuff out and post some updates. Feel free to comment if I'm reading the JSON structure in a completely asinine way. The vector->list doesn't seem right.

## Update
The couchdb egg documentation has been updated to cover most of this. I've also noticed a generally handy function that is exported by the couchdb module, json-ref, that makes it simple to grab top-level JSON items like "version" above.

```
(define server-version (json-ref 'version (get-server-info cloudant)))
```