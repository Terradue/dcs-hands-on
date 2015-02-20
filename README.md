Developer Cloud Sandbox Hands-On Exercises
==========================================

Installation
-------------

* Log on your Developer Cloud Sandbox host

* Install the needed packages:

```bash
sudo yum install -y esa-beam-4.11 ImageMagick
```

* Run these commands in a shell:

```bash
cd ~
git clone git@github.com:Terradue/dcs-hands-on.git
cd dcs-hands-on
mvn clean install -Dhands.on=-id-
```

where *-id-* is the identified of the Hands On you want to install. For example:

```
mvn clean install -Dhands.on=1
```

Available Hands On
------------------

* [Hands-On Exercise 1: a basic workflow](src/main/app-resources/hands-on-1)
* [Hands-On Exercise 2: make a robust workflow and debug it](src/main/app-resources/hands-on-2)
* [Hands-On Exercise 3: staging data](src/main/app-resources/hands-on-3)
* [Hands-On Exercise 4: using a toolbox](src/main/app-resources/hands-on-4)
* [Hands-On Exercise 5: using parameters](src/main/app-resources/hands-on-5)
* [Hands-On Exercise 6: multi-node workflow](src/main/app-resources/hands-on-6)
* [Hands-On Exercise 7: debug a multi-node workflow](src/main/app-resources/hands-on-7)
* [Hands-On Exercise 8: browse published results](src/main/app-resources/hands-on-8)
* [Hands-On Exercise 9: using an OpenSearch catalogue](src/main/app-resources/hands-on-9)
* [Hands-On Exercise 10: prepare an OGC Web Processing Service](src/main/app-resources/hands-on-10)
