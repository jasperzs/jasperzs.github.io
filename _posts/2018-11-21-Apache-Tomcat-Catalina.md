---
layout:     post
title:      Meet Apache Tomcat Catalina
subtitle:   Brief introduction to Apache Tomcat Catalina
date:       2018-11-21
author:     Mr.Humorous 🥘
header-img: img/apache/header.jpg
catalog: true
tags:
    - Apache
    - Tomcat
---

## 1. Meet Tomcat Catalina
Apache Tomcat is a widely used implementation of the Java Servlet Specification, which composes a number of components, including a Tomcat JSP engine and a variety of different connectors, but its core component is called Catalina. Catalina provides Tomcat's actual implementation of the servlet specification; when you start up your Tomcat server, you're actually starting Catalina.

## 2. Catalina's Configuration Files
Catalina's default behavior can be directly configured by editing the six configuration files located in Tomcat's `"$CATALINA_BASE/conf"` directory. Here's an overview of the files located in this directory and the kinds of options that can be configured within each.

_catalina.policy_<br>
This file contains the Tomcat security policy for the Catalina Java class, expressed in standard Security Policy syntax, as defined in the JEE specification. This is Tomcat's core security policy, and includes permissions definitions for system code, web applications, and Catalina itself.

_catalina.properties_<br>
This file is a standard Java properties file for the Catalina class. It contains information such as security package lists and class loader paths. This file can also contains some String cache settings, that you might edit when tuning your server for best Tomcat performance.

_logging.properties_<br>
This file configures the way that Catalina's built-in logging functions, including things such as threshold and log location. Note that all the entries in this log refer to JULI, the modified commons-logging implementation that Tomcat automatically uses in place of your JDK's logging implementation.

_content.xml_<br>
This XML configuration file is used to define Tomcat Context information that will be loaded for every web application running on a given instance of Tomcat. In general, you should configure your Context information elsewhere, but there are a few entries in this file that can be uncommented to alter the way that Tomcat handles session persistence and Comet connections.

_server.xml_<br>
This is Tomcat's main configuration file, which uses the hierarchical syntax specified in the Java Servlet specification to configure Catalina's initial state, as well as define the order in which Tomcat boots and builds its various components. This file is quite complex, but comprehensive documentation is available on the Apache website.

_tomcat-users.xml_<br>
This file contains information about the various users, passwords, and user roles on a given Tomcat server, as well as information about trusted Realms (JNDI, JDBC, etc.) where this data can be accessed.

_web.xml_<br>
This file configures options and values that will be applied to all applications loaded into a given instance of Tomcat, including servlet definitions such as buffer sizes, debugging levels, Jasper options like classpath, MIME types, and default welcome files for directories that do not have their own "index" files. Although you can technically configure options for specific web applications in this file, this will require you to restart your entire server to propagate these changes, so it is not recommended.

## 3. Rotating Catalina.out
A common question asked by Tomcat beginners is how to rotate the Catalina.out log file, without restarting Tomcat. (If you're not familiar with the file, Catalina.out is the standard destination log file for System.out and System.err, JVM loggers that print information about fatal errors in the JVM.)

There are two answers.

The first, which is more direct, is that you can rotate Catalina.out by adding a simple pipe to the log rotation tool of your choice in Catalina's startup shell script. This will look something like:

`"$CATALINA_BASE"/logs/catalina.out WeaponOfChoice 2>&1 &`

Simply replace "WeaponOfChoice" with your favorite log rotation tool.

The second answer is less direct, but ultimately better. The best way to handle the rotation of Catalina.out is to make sure it never needs to rotate. Simply set the "swallowOutput" property to true for all Contexts in "server.xml".

This will route System.err and System.out to whatever Logging implementation you have configured, or JULI, if you haven't configured it. All commons-logger implementations rotate logs by default, so rotating Catalina.out is no longer your problem.

## 4. Managing Catalina as an MBean
Because Catalina is a Java class, if you enable Java Management Extensions (JMX) management, you can actually manage all of Catalina's exposed functions as a single MBean, and reference all its hierarchical elements by name. Apache maintains a list of all the MBean names as part of the Tomcat documentation.

In order to begin managing Catalina as an MBean, all you need to do is modify the CATALINA_OPTS system variable to allow JMX access.
