---
layout:     post
title:      Directory Structure of Java Web Application
subtitle:   Reveal standard directory structure of Java Web App
date:       2019-03-31
author:     Mr.Humorous 🥘
header-img: img/webApp/header.jpg
catalog: true
tags:
    - Java
    - WebApp
---

## Java Web Application Directory Layout
In order to run your java web application(servlet,JSP etc) we need web server.Before running the application you need to package the resources inside it (servlets, JSP's,xml files etc.) in a standardized way as shown below.
![Directory Structure of Web App Illustration](/img/webApp/directoryStructureOfWebApp/directoryStructureOfWebApp.gif)

## The Root Directory
The root directory of you web application can have any name. In the above example the root directory name is mywebapp.

Under the root directory, you can put all files that should be accessible in your web application.

If your web application is mapped to the URL _http://localhost:8080/mywebapp/_ then the index.html page will be accessible by the URL _http://localhost:8080/mywebapp/index.html_

If you create any subdirectories under the root directory, and place files in these subdirectories, these will be available by the subdirectory/file path, in the web application. For instance, if you create a subdirectory called pages, and put a file _register.jsp_ inside it, then you could access that file from the outside, via this URL: _http://localhost:8080/mywebapp/pages/register.jsp_

## The WEB-INF Directory
The WEB-INF directory is located just below the web app root directory. This directory is a meta information directory. Files stored here are not supposed to be accessible from a browser (although your web app can access them internally, in your code).

Inside the WEB-INF directory there are two important directories (classes and lib, and one important file (web.xml). These are described below.

### web.xml
The web.xml file contains information about the web application, which is used by the Java web server / servlet container in order to properly deploy and execute the web application. For instance, the web.xml contains information about which servlets a web application should deploy, and what URL's they should be mapped to.

### classes Directory
The classes directory contains all compiled Java classes that are part of your web application. The classes should be located in a directory structure matching their package structure.

### lib folder
The lib directory contains all JAR files used by your web application. This directory most often contains any third party libraries that your application is using. You could, however, also put your own classes into a JAR file, and locate it here, rather than putting those classes in the classes directory.We will discuss more about this in next tutorial.

Instead of copying entire directory into server we can package them into war file and then do the deployment.

## What is war file
A WAR (or "web archive") file is simply a packaged webapp directory or archive file(like zip,rar etc) .It will store jsp,servlets,classes,meta data information,images,Sound and tag libararies etc. Its standard file extension is .war. WAR files are used to package Web modules. A WAR file is for a Web application deployed to a server. For example, an application for Stock trading might be distributed in an archive file called Stocktrading.war file.

The WAR file is a standard format for web applications that has specific directories and specific files. This includes a WEB-INF directory, a WEB-INF/web.xml file used to describe the application, a WEB-INF/lib directory for JAR files used by the application, and a WEB-INF/classes directory for class files that aren't distributed in a JAR. You would put the pages (JSPs and HTML) in the WAR as well. Then, you can distribute your application as one file, instead of as a collection of images, HTML pages, and Java classes.
