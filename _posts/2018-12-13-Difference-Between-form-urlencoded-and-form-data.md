---
layout:     post
title:      Difference between application/x-www-form-urlencoded and multipart/form-data in HTTP/HTML?
subtitle:   Understanding x-www-form-urlencoded and multipart/form-data
date:       2018-12-13
author:     Mr.Humorous 🥘
header-img: img/webApp/header.jpg
catalog: true
tags:
    - HTML
    - HTTP
    - Form Data
---

## 1. General Overview

Recently in one of the Java web developer interview, one of my readers asked about the __difference between x-www-form-url-encoded and multipart/form-data__ MIME types. In HTTP, there are two ways to send the HTML form data to the server either by using ContentType _application/x-www-form-urlencoded_ or by using _multipart/form-data_. Even though both can be used to send both text and binary data to the server there is a subtle difference between them. In the case of _x-www-form-urlencoded_, the whole form data is sent as a long query string.

The query string contains name-value pairs separated by & character e.g. _field1=value1&field2=value2_ etc. It is similar to URL encoding and normal GET request where data is sent on URL, but form data goes inside POST request body and they are encoded like that.

Also, both reserved and non-alphanumeric characters are replaced by '_%HH_', a percent sign and two hexadecimal digits representing the ASCII code of the character e.g. space is replaced by _%20_ character in URL.

On the other hand, when you choose HTTP header _ContentType=multipart/form-data_ then data is sent in chunks to a server where the boundary is made by a character which should not appear in the content.

This is achieved by using a suitable encoding e.g. choosing a base64 encoding and then making a character outside of base64 encoding scheme as the boundary. The multipart/form-data is often used while uploading files to the server.

## 2. x-www-form-urlencoded vs multipart/form-data

1. Both are MIME type which is sent on HTTP header ContentType e.g.
   - _ContentType: application/x-www-form-urlencoded_
   - _ContentType: application/multipart/form-data_
2. Both are ways to send name-value pairs data to the server i.e. the details you have entered into an HTML form is sent by using these two to the server.
3. Both content types are used while sending form data as a POST request.
4. The __x-www-form-urlencoded__ is used more generally to send text data to the server while _multipart/form-data_ is used to send binary data, most notably for uploading files to the server.
5. It's a requirement for user agents like a browser to support both MIME types.
6. In the case of _x-www-form-urlencoded_, all name value pairs are sent as one big query string where alphanumeric and reserved character are url encoded i.e. replaced by % and their hex value e.g. space is replaced by %20. The length of this string is not specified by HTTP specification and depends upon server implementation.
7. In the case of _multipart/form-data_, each part is separated by a particular string boundary (chosen specifically so that this boundary string does not occur in any of the "value" payloads.
8. The __multipart/form-data__ is more efficient than __x-www-form-urlencoded__ because you don't need to replace one character with three bytes as required by URL encoding.

## 3. Should you use multipart/form-data Always?

Given the _multipart/form-data_ is more efficient than _x-www-form-urlencoded_, some of you may question Why not use multipart/form-data all the time? Well, it's not the idea.

For short alphanumeric values (like most of the web forms), the overhead of adding all of the MIME headers is going to significantly outweigh any savings you will make from more efficient binary encoding.

Hence, it is advised to use x-www-form-urlencoded when you have to send form data e.g. most of the web form which asks you to enter values and use multipart/form-data when you have to upload files to the server.
![Multipart Form Data Example]({{ site.url }}/img/webApp/formData/multipartFormData.png)

That's all about the __difference between x-www-form-urlencoded and multipart/form-data__ content type headers in HTTP. Even though both are used to send the form data or a list of key-value pairs to the server, x-www-form-urlencoded is more efficient and should be used for all general purpose, while multipart/form-data should exclusively be used for uploading files to the server.
