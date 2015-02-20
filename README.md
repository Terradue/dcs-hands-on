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
git clone git@github.com:Terradue/dcs-handson.git
cd dcs-testsuite
mvn clean install -Ddcs.test.id=id -P bash
```

where *id* is the Hands On id you want to install. For example:

```
mvn clean install -Ddcs.handson.id=1 -P bash
```

Available Hands On
------------------

* [Hands-On Exercise 1: a basic workflow](src/main/app-resources/hands-on-1)
