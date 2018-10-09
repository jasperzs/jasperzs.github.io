---
layout:     post
title:      What is Blobs and File Interface
subtitle:   Demonstrate Blobs and File Interface
date:       2018-10-11
author:     Mr.Humorous 🥘
header-img: img/javaScript/header.png
catalog: true
tags:
    - JavaScript
    - Blob
    - File
---

## 1. What is a Blob?
A blob object represents a chuck of bytes that holds data of a file, but it is not a reference to a actual file.
It can be very large which can contain audio and video data. Also, it can be created dynamically and using blob URLs they can be used as files.

A blob has its size and MIME type just like a file has. Blob data is stored in the memory or filesystem depending on the browser and blob size.
A blob can be used like a file wherever we use files.

Blobs often work with asynchronous APIs, but they can also work with synchronous ones in the case of Web Workers.

## 2. Creating and Reading From a Blob
```javascript
//First arguement must be a regular array of any javascript objects. Array can contain array to make it multi dimensional.
//Second parameter must be a BlogPropertyBag object containing MIME property
var myBlob = new Blob(["This is my blob content"], {type : "text/plain"});
var myReader = new FileReader();
//handler executed once reading(blob content referenced to a variable) from blob is finished.
myReader.addEventListener("loadend", function(e){
    document.getElementById("paragraph").innerHTML = e.srcElement.result;//prints a string
});
//start the reading process.
myReader.readAsText(myBlob);
```

## 3. Blob URLs
blob:// URLs reference to a blob which can be used almost wherever regular URLs are used.
```javascript
//cross browser
window.URL = window.URL || window.webkitURL;

var blob = new Blob(['body { background-color: yellow; }'], {type: 'text/css'});

var link = document.createElement('link');
link.rel = 'stylesheet';
//createObjectURL returns a blob URL as a string.
link.href = window.URL.createObjectURL(blob);
document.body.appendChild(link);
```

## 4. Remote data as Blobs

### 4.1 Without FileReader.readAsArrayBuffer()
```javascript
var xhr = new XMLHttpRequest();
xhr.open("GET", "/favicon.png");
xhr.responseType = "blob";//force the HTTP response, response-type header to be blob
xhr.onload = function()
{
    document.getElementsByTagName("body")[0].innerHTML = xhr.response;//xhr.response is now a blob object
}
xhr.send();
```

### 4.2 With FileReader.readAsArrayBuffer()
```javascript
var xhr = new XMLHttpRequest();
xhr.open("GET", "/favicon.png");
//although we can get the remote data directly into an arraybuffer using the string "arraybuffer" assigned to responseType property. For the sake of example we are putting it into a blob and then copying the blob data into an arraybuffer.
xhr.responseType = "blob";

function analyze_data(blob)
{
    var myReader = new FileReader();
    myReader.readAsArrayBuffer(blob)

    myReader.addEventListener("loadend", function(e)
    {
        var buffer = e.srcElement.result;//arraybuffer object
    });
}

xhr.onload = function()
{
    analyze_data(xhr.response);
}
xhr.send();
```

## 5. File Interface
A File object in JavaScript references an actual file in the local filesystem and there is no way to create it.
It inherits all properties and methods from the Blob class and thus they expose same methods and properties.
```javascript
var element = document.getElementsByTagName("body")[0];

//files is a filelist
function fileselected(files)
{
    for(var i = 0; i < files.length; i++)
    {
        var f = files[i];
        element.innerHTML = element.innerHTML + f.name + " " + f.size + " " + f.type;
    }
}
```
