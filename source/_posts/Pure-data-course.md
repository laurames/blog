---
title: Pure data course
date: 2016-09-17 18:28:13
description: 'This is a project to produce music by using the wikipedia live edit stream at: stream.wikimedia.org. To access the stream Socket.IO 0.9 protocol is used by installing it with npm. For the music I am using Matt Davey pure data abstractions and creating different sounds depending on the first ascii character of "title" received from the real-time Wikipedia edits stream.'
categories:
  - aalto
tags:
  - aalto
  - wikipedia
  - live stream
  - courses
  - coding
---
---
{% asset_img wikipediaStream2.png Wikipedia live stream with pure data producing sounds %}

# Table of Contents
1. [The Course: Composing with Data Flow Programming](#The-Course-Composing-with-Data-Flow-Programming)
  1. [What is Pure Data](#What-is-Pure-Data)
2. [My Project: Real Time Wikipedia With PureData](#My-Project-Real-Time-Wikipedia-With-PureData)
  1. [The Requirements](#The-Requirements)
  2. [Socket.io](#Socket.io)
  3. [Connecting to Pure Data](#Connecting-to-Pure-Data)
  4. [Connecting to Processing](#Connecting-to-Processing)

## The Course: Composing with Data Flow Programming

* **Status of the Course**
  Ma in new media, Sound, optional
* **Level of the Course**
  advanced
* **Teacher in charge**
  Koray Tahiroglu
* **Teaching Period** 	
  0 - I (fall2016)
* **Workload** 	
  This is a project-based course, at the end of the course students will submit and present their group or individual projects. 70 h teaching.
* **Learning Outcomes**
  Composing with Data Flow programing will introduce students to Pure Data (a.k.a Pd) - a real-time graphical data flow programing environment for audio, video and graphical processing. See http://puredata.info/ for more information.  Students will learn the basics of using a data flow programming environment, simple audio synthesis techniques, audio sample playback, basic video manipulation, and methods of connecting external interfaces, including how to use networking tools, and using them as methods of control.
* **Content**
  Course provides the opportunity to learn the basic skills and tools to create, process and organise sounds, video processing and map processes to physical devices/sensors.  This is a project-based course; at the end of the course students will submit and present their group or individual projects.
* **Assessment Methods and Criteria**
  The course consists of lectures, exercises, reading materials, tutoring individual or group works. Students will submit their documented project work and ~ 750 words learning diary, both grounds the course examination and final grade. Each student project work will be assessed with the following criteria: Design Values, Aesthetics and Originality; UI design and Production Values; Code Design Quality; Project Analysis - Depth of Understanding; Idea generation and implementation; and Presentation style.
* **Course Homepage**
  [Composing with Data Flow Programming – Autumn 2016](https://sopi.aalto.fi/teaching/cwdfp/2768-2/)

### What is Pure Data

> Pure Data (or Pd) is a real-time graphical programming environment for audio, video, and graphical processing. Pure Data is commonly used for live music performance, VeeJaying, sound effects, composition, audio analysis, interfacing with sensors, using cameras, controlling robots or even interacting with websites. [flossmanuals Pure Data](http://en.flossmanuals.net/pure-data/)

Pd is a data flow programming language and supports four basic types of text entities: messages, objects, atoms, and comments. Atoms consist of either a float, a symbol, or a pointer to a data structure and are the most basic unit of data in Pd. In Pd, all numbers are stored as 32-bit floats. Messages provide instructions to objects and are composed of one or more atoms. To initiate events and push data into flow, much like pushing a button, a special type of message with null content called a bang is used.

> Pure Data (Pd) is a visual programming language developed by Miller Puckette in the 1990s for creating interactive computer music and multimedia works. While Puckette is the main author of the program, Pd is an open source project with a large developer base working on new extensions. It is released under a license similar to the BSD license. It runs on GNU/Linux, Mac OS X, iOS, Android and Windows. Ports exist for FreeBSD and IRIX. [wikipedia Pure Data](https://en.wikipedia.org/wiki/Pure_Data)

You can download Pd from [here](http://puredata.info/downloads). The learning curve is quite steep, but once you pick up and **remember by heart the graphic syntax** and know a lot about **audio composing methods** and **science behind producing sound** you're good to go for Pd. Personally I struggled a lot to grasp the idea behind it.

## My Project: Real Time Wikipedia With PureData

{% asset_img pureData.png My main pure data pad %}

First of all the code for this project can all be found [here](https://github.com/laurames/realTimeWikipediaWithPureData). It includes all the pure data pads used, socket.io code for running both the server and the wikipedia stream. There is also some extra things that i needed to submit in order to pass this course, but are in no way needed to run this project on your own.

### The Requirements

**It is important you have an older version (5) of [node](https://nodejs.org/en/) installed! Don't install the newest one!**

**As of January 2015, RCStream implements version 0.9 of the Socket.IO protocol, not 1.0 (phab:T68232). See also [socket.io 0.9](https://github.com/socketio/socket.io/tree/0.9.17) and [socket.io-client 0.9](https://github.com/socketio/socket.io-client/tree/0.9.17) on GitHub for more information.**

The wikipedia recent changes stream info can be found [here](https://www.mediawiki.org/wiki/API:Recent_changes_stream). When starting the project i was able to get access to the stream but getting something from the stream to pure data was a challenge.

### Socket.io

First I needed to create a **local server** so that I could have access to the wiki stream from pure data. This is because pure data only supports two ways of accessing external data. One is [netsend] and [netreceive] that are objects for transmitting and receiving messages over a network. The second is Open Sound Control (OSC) and these are objects for sharing musical data over a network. "Using OSC you can exchange data with a number of devices, such as Lemur, iPhone (through OSCulator), Monome, or applications such as Ardour, Modul8, Reaktor and many more. Most modern programming languages are OSC enabled, notably Processing, Java, Python, C++, Max/MSP and SuperCollider." [Open Sound Control (OSC)](http://en.flossmanuals.net/pure-data/network-data/osc/). When we send the data to the netsend port I make sure i send only the first letter of the title that we get from the wikipedia stream.

``` javaScript
// port setup
var pd = port({
	'read': 8005, // [netsend]
	'write': 8006, // [netreceive]
  'encoding': 'utf-8',
	'flags': {
		'noprefs': true, // '-stderr', '-nogui',
    'open': 'patches/main.pd' //opens the file
	}
})
.on('stderr', function(buffer){
	console.log(buffer.toString());
})
.create();

var HOST = '127.0.0.1';
var PORT = 3000;

// Create a server instance, and chain the listen function to it
// The function passed to net.createServer() becomes the event handler for the 'connection' event
// The sock object the callback function receives UNIQUE for each connection
net.createServer(function(sock) {

  // We have a connection - a socket object is assigned to the connection automatically
  console.log('CONNECTED: ' + sock.remoteAddress +':'+ sock.remotePort);

  // Add a 'data' event handler to this instance of socket
  sock.on('data', function(data) {
    console.log(data.toString('utf-8'));
    pd.write(data.toString('utf-8').charAt(0) + ';\n');
		//pd.write(data.toString('utf-8').charAt(0) + ' ' + data.toString('utf-8').charAt(1) + ';\n');
  });

  // Add a 'close' event handler to this instance of socket
  sock.on('close', function(data) {
    console.log('CLOSED: ' + sock.remoteAddress +' '+ sock.remotePort);
  });

  sock.on('error', function(data) {
    console.log('error has accured');
  });

}).listen(PORT, HOST);
```

Next we create the actual **wiki stream** so that we can connect it to the opened server port. Here we just connect to the provided RC Stream and when there is a change in data we feed the title of the change.

``` javaScript
var io = require( 'socket.io-client' );
var socket = io.connect( 'https://stream.wikimedia.org/rc' ); //, {'sync disconnect on unload': true }
console.log('check if connected to stream: ', socket.connected);

const net = require('net');

const client = net.connect({port: 3000}, () => {
  // 'connect' listener
  console.log('Connected to server at 3000');
});

//full RCFeed properties can be found at: https://www.mediawiki.org/wiki/Manual:RCFeed#Properties
socket.on( 'change', function ( data ) {
  console.log( data.title ); // this is the edited title
  //console.log( data.length.new ); // this is the edited length
  const buf = Buffer.from(data.title, 'utf-8');
  client.write(buf);
} );
```

### Connecting to Pure Data

In Pure Data using the [netsend] and [netreceive] I access the port 8006 to read the data and covert the input to ascii values using the object [spell]. Then with [iter] we split a list of ascii codes into a series of numbers. Depending on this number we are able to produce different sounds. For the sounds i'm using Matt Davey's [DIY patches](https://github.com/derekxkwan/pure-data-abstractions). To get these onto a personal Pd project one has to copy the necessary patches completely (Check picture below).

{% asset_img wikipediaStream3.png One patch using Matt Davey's DIY patches %}

### Connecting to Processing

Lastly I wanted to connect what ever music was produced into Processing to make graphics out of the live stream data. I was able to make the connection using OSC, but time ran out to actually create any cool graphics with the data i was sending with Pd. I'll just leave you with some inspiration before i can come back to making mine awesome too.

{% asset_img Collage_Processing.jpg Connecting to Processing with OSC %}

Some great Processing examples and inspiration for future projects by [Joshua Davis](http://www.joshuadavis.com/). He also created [Hype](http://www.hypeframework.org/) library with James Cruz. It is a collection of classes that performs heavy lifting tasks while using a minimal amount of code writing using Processing and ProcessingJS.

{% vimeo 167338762 %}
