# m2m

[![Version npm](https://img.shields.io/npm/v/m2m.svg?logo=npm)](https://www.npmjs.com/package/m2m)
![Custom badge](https://img.shields.io/endpoint?url=https%3A%2F%2Fwww.node-m2m.com%2Fm2m%2Fbuild-badge%2F2021)

m2m is a client module for machine-to-machine communication framework  [node-m2m](https://www.node-m2m.com).

The module's API is a FaaS (Function as a Service) also called "serverless" making it easy for everyone to develop applications in telematics, data acquisition, process automation, network gateways, workflow orchestration and many others.

You can deploy multiple public device servers on the fly from anywhere without the usual heavy infrastructure involved in provisioning a public server.

You can set multiple *channel data*, *GPIO objects* and *HTTP API* as server resources.

Your device servers will be accessible through its user assigned *device id* from client applications.

Access to clients and devices is restricted to authenticated and authorized users only.

All communications between client and device servers are fully encrypted using TLS protocol.

To use this module, users must create an account and register their devices with [node-m2m](https://www.node-m2m.com).

[](https://raw.githubusercontent.com/EdoLabs/src/master/m2mSystem2.svg?sanitize=true)

# Table of contents
1. [Supported Platform](#supported-platform)
2. [Node.js version requirement](#nodejs-version-requirement)
3. [Installation](#installation)
4. [Quick Tour](#quick-tour)
5. [Examples](#examples)
   * [Using MCP 9808 Temperature Sensor](#using-mcp-9808-temperature-sensor)
   * [GPIO Input Monitoring and Output Control](#gpio-input-monitoring-and-output-control)
   * [Remote Machine Control](#remote-machine-control)
   * [Using Channel Data for GPIO Control](#using-channel-data-for-gpio-control)
   * [Sending Data to Remote Device](#sending-data-to-remote-device)
6. [HTTP API](#http-api)
    * [Server GET and POST method Setup](#server-get-and-post-method)
    * [Client GET and POST request](#client-get-and-post-request)
7. [Using The Browser Interface To Access Clients and Devices](#using-the-browser-interface-to-access-clients-and-devices)
   * [Naming Your Client Application for Tracking Purposes](#naming-your-client-application-for-tracking-purposes)
   * [Remote Code Editing](#remote-application-code-editing)
   * [Application Process Auto Restart](#application-auto-restart)
   * [Configure Your Application Process for Remote Code Editing and Auto Restart](#code-edit-and-auto-restart-automatic-configuration)
8. [Node-M2M Server Query](#node-m2m-server-query)
   * [Server query to get all available remote devices](#server-query-to-get-all-available-remote-devices)
   * [Server query to get each device resources](#server-query-to-get-each-device-resources)


## Supported Platform

* Raspberry Pi Models: B+, 2, 3, Zero & Zero W, Compute Module 3, 3B+, 3A+, 4B (generally all 40-pin models)
* Linux
* Windows
* Mac OS

## Node.js version requirement

* Node.js versions: 8.x, 9.x, 10.x, 11.x, 12.x, 14.x

## Installation
```js
$ npm install m2m
```

![]()
###  Raspberry Pi peripheral access (GPIO, I2C, SPI and PWM). <a name="rpi-peripheral-access"></a>
For projects requiring raspberry pi peripheral access such as GPIO, I2C, SPI and PWM, you will need to install *array-gpio* as a separate module.
```js
$ npm install array-gpio
```

![]()
## Quick Tour

We will create a server (*remote device*) that will generate random numbers as its sole service.

And a client application (*remote client*) that will access the random numbers.

The client will access the random numbers using a pull and push method.

Using a *pull-method*, the client will capture the random numbers using a one time function call.

Using a *push-method*, the client will watch the random numbers every 5 seconds for any changes. If the value changes, the remote device will send the new value to the remote client.   

![](https://raw.githubusercontent.com/EdoLabs/src2/master/quicktour.svg?sanitize=true)
[](quicktour.svg)

Before you start, create an account and register your remote device with node-m2m server.

### Remote Device Setup

Create a device project directory and install m2m.
```js
$ npm install m2m
```
Create the file below as device.js within your device project directory.
```js
const m2m = require('m2m');

// create a device object with a device id of 100
// this id must must be registered with node-m2m
let device = new m2m.Device(100);

device.connect(function(err, result){
  if(err) return console.error('connect error:', err);

  console.log('result:', result);

  // set 'random' channel as resource  
  device.setData('random', function(err, data){
    if(err) return console.error('setData random error:', err.message);

    let rd = Math.floor(Math.random() * 100);
    data.send(rd);
  });
});
```

Start your device application.
```js
$ node device.js
```

The first time you run your application, it will ask for your full credentials.
```js
? Enter your userid (email):
? Enter your password:
? Enter your security code:

```
The next time you run your application, it will start automatically using a saved user token.

However, when your application is already running for more than 15 minutes, your application becomes immutable. Any changes to your application code will require you to re-authenticate to restart your application for security reason.

Restart your application using `$ node device.js` or using the CLI below.

At anytime, you can re-authenticate with full credentials using the *-r* flag when restarting your application as shown below.

```js
$ node device.js -r
```
### Remote Client Setup
Similar with the remote device setup, create a client project directory and install m2m.
```js
$ npm install m2m
```
Create the file below as client.js within your client project directory.

#### Accessing resources from your remote device
To access resources from your remote device, create a device object using the client's *accessDevice* method as shown below. The device object created as expected represents a specific remote device as indicated by the *device id* argument. In this tour, the device id is the integer value **100**;

The device object provides various methods to access channel data, GPIO data and HTTP API resources from your remote device servers.

```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(err, result){
  if(err) return console.error('connect error:', err);

  console.log('result:', result);

  // create a remote device object from client accessDevice method
  let device = client.accessDevice(100);

  // capture 'random' data using a one-time function call
  device.getData('random', function(err, data){
    if(err) return console.error('getData random error:', err.message);
    console.log('random data', data); // 97
  });

  // capture 'random' data using a push method
  // the remote device will scan/poll the data every 5 secs (default)
  // if the value changes, it will push/send the data to the client
  device.watch('random', function(err, data){
    if(err) return console.error('watch random error:', err.message);
    console.log('watch random data', data); // 81, 68, 115 ...
  });
});
```
Start your application.
```js
$ node client.js
```
Similar with remote device setup, you will be prompted to enter your credentials.

You should get a similar output result as shown below.
```js
random data 97
watch random data 81
watch random data 68
watch random data 115
```
## Examples

### Using MCP 9808 Temperature Sensor

![](https://raw.githubusercontent.com/EdoLabs/src2/master/example1.svg?sanitize=true)
[](example1.svg)
#### Remote Device Setup in Tokyo

```js
$ npm install m2m array-gpio
```
```js
const m2m = require('m2m');

// using a built-in MCP9808 i2c library using array-gpio
// you can also create your own 9808 library using other npm modules
const i2c = require('./node_modules/m2m/examples/i2c/9808.js');

let device = new m2m.Device(110);

// explicitly connecting to default node-m2m server
device.connect('https://www.node-m2m.com', function(err, result){
  // device application logic
});

// or

// implicitly connecting to default node-m2m server
device.connect(function(err, result){
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  device.setData('temperature', function(err, data){
    if(err) return console.error('set temperature error:', err.message);

    let td =  i2c.getTemp();
    data.send(td);
  });
});
```

#### Client application in Boston
```js
$ npm install m2m
```
```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(err, result){
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  let device = client.accessDevice(110);

  device.watch('temperature', function(err, data){
    if(err) return console.error('temperature error:', err.message);
    console.log('temperature data', data); // 23.51, 23.49, 23.11
  });
});
```
#### Client application in London
```js
$ npm install m2m
```

```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(err, result){
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  let device = client.accessDevice(110);

  // scan/poll the data every 15 secs instead of the default 5 secs
  device.watch('temperature', 15000, function(err, data){
    if(err) return console.error('temperature error:', err.message);
    console.log('temperature data', data); // 23.51, 23.49, 23.11
  });

  // unwatch temperature data after 5 minutes
  // client will stop receiving temperature data from the remote device
  setTimeout(function(){
    device.unwatch('temperature');
  }, 5*60000);

  // watch temperature data again after 10 minutes
  // since no scan/poll interval argument was provided, it will scan the data every 5 secs (default)
  // client will start receiving again the temperature data from the remote device
  setTimeout(function(){
    device.watch('temperature');
  }, 10*60000);

});
```

### GPIO Input Monitoring and Output Control

![](https://raw.githubusercontent.com/EdoLabs/src2/master/example2.svg?sanitize=true)

#### Configure GPIO input as resource on device1
Install array-gpio both on device1 and device2
```js
$ npm install m2m array-gpio
```

```js
const m2m = require('m2m');

let device = new m2m.Device(120);

device.connect(function(err, result){
  if(err) return console.error('connect error:', err);

  console.log('result:', result);

  // set GPIO input as resource
  device.setGpio({mode:'input', pin:[11, 13]}, function(err, gpio){
    if(err) return console.error('setGpio input error:', err.message);

    console.log('input pin', gpio.pin, 'state', gpio.state);
    // you can provide additional custom logic here
  });
});
```

#### Configure GPIO output as resource on device2
```js
$ npm install m2m array-gpio
```
```js
const m2m = require('m2m');

let device = new m2m.Device(130);

device.connect(function(err, result){
  if(err) return console.error('connect error:', err);

  console.log('result:', result);

  // set GPIO output as resource
  device.setGpio({mode:'output', pin:[33, 35]}, function(err, gpio){
    if(err) return console.error('setGpio output error:', err.message);

    console.log('output pin', gpio.pin, 'state', gpio.state);
    // you can provide additional custom logic here
  });
});
```
#### Client accessing GPIO input/output resources from device1 and device2
```js
$ npm install m2m
```
There are two ways we can access the GPIO input/output objects from remote devices as shown below.
Both ways will allow you to manipulate GPIO input/output resources directly from client applications.  
```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(err, result){
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  let device1 = client.accessDevice(120);
  let device2 = client.accessDevice(130);

  /* using .gpio() method */
  // get current state of device1 input pin 11
  device1.gpio({mode:'in', pin:11}).getState(function(err, state){
    if(err) return console.error('get input pin 11 state error:', err.message);
    console.log('get input pin 11 state', state);
  });

  // watch device1 input pin 11 for state changes every 5 secs
  device1.gpio({mode:'in', pin:11}).watch(function(err, state){
    if(err) return console.error('watch input pin 13 state error:', err.message);
    console.log('watch input pin 11 state', state);

    if(state){
      // turn ON output pin 33
      device2.gpio({mode:'out', pin:33}).on();
    }
    else{
      // turn OFF output pin 33
      device2.gpio({mode:'out', pin:33}).off();
    }
  });

  /* using .input()/output() method */
  // get current state of device1 input pin 13
  device1.input(13).getState(function(err, state){
    if(err) return console.error('get input pin 13 state error:', err.message);
    console.log('get input pin 13 state', state);
  });

  // watch device1 input pin 13 for state changes every 5 secs
  device1.input(13).watch(function(err, state){
    if(err) return console.error('watch pin 11 error:', err.message);
    console.log('watch input pin 13 state', state);

    if(state){
      // turn OFF output pin 35
      device2.output(35).off();
    }
    else{
      // turn ON output pin 35
      device2.output(35).on();
    }
  });
});
```
### Using Channel Data for GPIO Control

Aside from the available standard api for Raspberry Pi direct GPIO control, we can also use the channel data api to manipulate and control GPIO input/output resources from any Raspberry Pi devices.

If you notice, the standard gpio api requires us to use *array-gpio* as dependency.

With channel data api, we can use other *npm modules* for Raspberry Pi GPIO control.

### Example 1
#### Device/Server Raspberry Pi setup
```js
const m2m = require('m2m');
const { setOutput } = require('array-gpio');

const led = setOutput(33);

let server = new m2m.Device(200);

server.connect(function(err, result){
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  // 'gpio-output-pin-33-off' channel data resource
  server.setData('gpio-output-pin-33-off', function(err, data){
    if(err) return console.error('gpio-output-pin-33-off error:', err.message);

    led.off();
    // send back a response
    data.send('led is off');
  });

  // 'gpio-output-pin-33-on' channel data resource
  server.setData('gpio-output-pin-33-on', function(err, data){
    if(err) return console.error('gpio-output-pin-33-on error:', err.message);

    led.on();
    // send back a response
    data.send('led is on');
  });
});
```

#### Client application to turn ON and OFF the remote Raspberry Pi's led using channel data
```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(err, result){
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  let device = client.accessDevice(200);

  device.getData('gpio-output-pin-33-on', function(err, data){
    if(err) return console.error('gpio-output-pin-33-on error:', err.message);

    console.log('gpio-output-pin-33-on', data); // 'led is on'
  });

  device.getData('gpio-output-pin-33-off', function(err, data){
    if(err) return console.error('gpio-output-pin-33-off error:', err.message);

    console.log('gpio-output-pin-33-off', data); // 'led is off'
  });      

});
```
### Example 2
#### Device/Server Raspberry Pi setup
```js
const m2m = require('m2m');
const { setOutput } = require('array-gpio');

const led = setOutput(33);

let server = new m2m.Device(200);

server.connect(function(err, result){
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  // 'gpio-output-pin-33-off' channel data resource
  server.setData('gpio-output-pin-33', function(err, data){
    if(err) return console.error('gpio-output-pin-33 error:', err.message);

    if(data.payload === 'on'){
      led.on();
      data.send('led is on');
    }
    else{
      led.off();
      data.send('led is off');
    }

  });
});  
```
#### Client application to turn ON and OFF the led using senData method
```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(err, result){
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  let device = client.accessDevice(200);

  device.sendData('gpio-output-pin-33', 'on', function(err, data){
    if(err) return console.error('gpio-output-pin-33 on error:', err.message);

    console.log('gpio-output-pin-33-on', data); // 'led is on'
  });

  device.sendData('gpio-output-pin-33', 'off', function(err, data){
    if(err) return console.error('gpio-output-pin-33 off error:', err.message);

    console.log('gpio-output-pin-33-off', data); // 'led is off'
  });      

});
```
### Remote Machine Control

![](https://raw.githubusercontent.com/EdoLabs/src2/master/example3.svg?sanitize=true)
[](example3.svg)

#### Configure each remote machine's rpi microcontroller with a unique device *id* <br> and set pin 40 for GPIO output control

Install array-gpio on all remote machines for GPIO output control
```js
$ npm install m2m array-gpio
```
```js
const m2m = require('m2m');

// assign 1001, 1002 and 1003 respectively for each remote machine
let device = new m2m.Device(1001);

device.connect((err, result) => {
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  // set pin 40 as GPIO output to activate the actuator
  device.setGpio({mode:'output', pin:40});
});
```

#### Client application to control the remote machines
```js
$ npm install m2m
```

```js
const m2m = require('m2m');

const client = new m2m.Client();

let interval;

client.connect((err, result) => {
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  // accessing multiple remote devices using an array object
  let devices = client.accessDevice([1001, 1002, 1003]);

  clearInterval(interval);
  machineControl(devices);
  // iterate over the remote machines every 15 secs
  interval = setInterval(machineControl, 15000, devices);

});

function machineControl(devices){
  let t = 0;    // initial output time delay
  let pin = 40; // actuator gpio output pin
  devices.forEach((device) => {
    t += 2000;

    // turn ON device gpio output pin 40
    device.output(pin).on(t, (err, state) => {
      if(err) return console.error('actuator ON err:', err.message);
      console.log('machine ', device.id, ' pin ', pin, ' output ON', state);

      // turn OFF device gpio output pin 40
      device.output(pin).off(t + 400, (err, state) => {
        if(err) return console.error('actuator OFF err:', err.message);
        console.log('machine ', device.id, ' pin ', pin, ' output OFF', state);
      });
    });
  });
}
```
### Sending Data to Remote Device
[](https://raw.githubusercontent.com/EdoLabs/src2/master/example4.svg?sanitize=true)
[](example1.svg)

#### Device/Server Setup
Instead of the usual capturing of data from remote devices, we can send data to remote devices for resource updates or as part of an application requirement.  

```js
const m2m = require('m2m');
const fs = require('fs');

let server = new m2m.Device(500);

server.connect(function(err, result){
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  // 'echo-server' channel data resource
  server.setData('echo-server', function(err, data){
    if(err) return console.error('echo-server error:', err.message);
    // send back the payload to client
    data.send(data.payload);
  });

  // 'send-file' channel data resource
  server.setData('send-file', function(err, data){
    if(err) return console.error('send-file error:', err.message);

    let file = data.payload;

    fs.writeFile('myFile.txt', file, function (err) {
      if (err) throw err;
      console.log('file has been saved!');
      // send a response
      data.send({result: 'success'});
    });
  });

  // 'send-data' channel data resource
  server.setData('send-data', function(err, data){
    if(err) return console.error('send-data error:', err.message);

    console.log('data.payload', data.payload);
    // data.payload  [{name:'Ed'}, {name:'Jim', age:30}, {name:'Kim', age:42, address:'Seoul, South Korea'}];

    // send a response
    if(Array.isArray(data.payload)){
      data.send({data: 'valid'});
    }
    else{
      data.send({data: 'invalid'});
    }
  });

  // 'number' channel data resource
  server.setData('number', function(err, data){
    if(err) return console.error('number error:', err.message);

    console.log('data.payload', data.payload); // 1.2456
  });

});
```

#### Client sending data to remote device
```js
const fs = require('fs');
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(err, result){
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  let server = client.accessDevice(500);

  // sending a simple string payload data to 'echo-server' channel
  let payload = 'hello server';

  server.sendData('echo-server', payload , function(err, data){
    if(err) return console.error('echo-server error:', err.message);

    console.log('echo-server', data); // 'hello server'
  });

  // sending a text file
  let myfile = fs.readFileSync('myFile.txt', 'utf8');

  server.sendData('send-file', myfile , function(err, data){
    if(err) return console.error('send-file error:', err.message);

    console.log('send-file', data); // {result: 'success'}
  });

  // sending a json data
  let mydata = [{name:'Ed'}, {name:'Jim', age:30}, {name:'Kim', age:42, address:'Seoul, South Korea'}];

  server.sendData('send-data', mydata , function(err, data){
    if(err) return console.error('send-data error:', err.message);

    console.log('send-data', data); // {data: 'valid'}
  });

  // sending data w/o a response
  let num = 1.2456;

  server.sendData('number', num);

});
```

## HTTP API

### Server GET and POST method
```js
const m2m = require('m2m');

const server = new m2m.Server(300);

server.connect((err, result) => {
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  let myData = {name:'Jim', age:34};

  // set a GET method resource
  server.get('data/current', (err, data) => {
    if(err) return console.error('data/current error:', err.message);
    // send current data
    data.send(myData);
  });

  // set a POST method resource
  server.post('data/update', (err, data) => {
    if(err) return console.error('data/update error:', err.message);

    myData = data.payload;
    // send a 'success' response
    data.send('success');
  });
});
```

### Client GET and POST request
```js
const m2m = require('m2m');

const client = new m2m.Client();

client.connect((err, result) => {
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  let server = client.accessServer(300);

  // GET method request
  server.get('data/current', (err, data) => {    
    if(err) return console.error('data/current error:', err.message);

    console.log('data/current', data); // {name:'Jim', age:34}
  });

  // POST method request
  server.post('data/update', {name:'ed', age:35} , (err, data) => {   
    if(err) return console.error('data/update error:', err.message);

    console.log('data/update', data); // 'success'
  });

  // get data after update
  server.get('data/current'); // data/current {name:'ed', age:35}
  // using the 'data/current' initial callback for the result

});
```
## Using The Browser Interface To Access Clients and Devices

### Naming Your Client Application for Tracking Purposes

Unlike *device/server* applications, users can create *client* applications without server registration.

Node-m2m tracks all client applications through a dynamic *client id* from the browser.
If you have multiple clients, tracking all your clients by just referring to its *client id* maybe difficult.

You can add a **name**, **location** and a **description** properties to your clients as shown below for tracking purposes.

This will make it easy to track all your clients from the browser interface.
```js
const m2m = require('m2m');

const client = new m2m.Client({name:'Main client', location:'Boston, MA', description:'Test client app'});

client.connect((err, result) => {
  if(err) return console.error('connect error:', err);
  console.log('result:', result);
  ...
});
```

### Remote Application Code Editing

Using the browser interface, you can download, edit and upload your application code from your remote clients and devices from anywhere.

To allow the browser to communicate with your application, add the following *m2mConfig* property to your project's package.json.

```js
"m2mConfig": {
  "code": {
    "allow": true,
    "filename": "device.js"
  }
}
```
Set the property *allow* to true and provide the *filename* of your application.

From the example above, the filename of the application is *device.js*. Replace it with the actual filename of your application.


### Application Auto Restart

Using the browser interface, you may need to restart your application after a module update, code edit/update, or by sending a remote restart command.

Node-m2m uses **nodemon** to restart your application.

You can add the following *nodemonConfig* and *scripts* properties in your project's npm package.json as *auto-restart configuration*.
```js
"nodemonConfig": {
  "delay":"2000",
  "verbose": true,
  "restartable": "rs",
  "ignore": [".git", "public"],
  "ignoreRoot": [".git", "public"],
  "execMap": {"js": "node"},
  "watch": ["node_modules/m2m/mon"],
  "ext": "js,json"
}
"scripts": {
  "start": "nodemon device.js"
},
```
From the example above, the filename of the application is *device.js*. Replace it with the actual filename of your application when adding the scripts property. Then restart your node process using *npm start* command as shown below.
```js
$ npm start
```
For other custom nodemon configuration, please read the nodemon documentation.

## Code Edit and Auto Restart Automatic Configuration
To automatically configure your package.json for code editing and auto restart without manually editing your package.json, start your node process with *-config* flag.

**m2m** will attempt to configure your package.json by adding/creating the *m2mConfig*, *nodemonConfig*, and *scripts* properties to your existing project's package.json. If your m2m project does not have an existing package.json, it will create a new one.  

Assuming your application filename is *device.js*, start your node application as shown below.
```js
$ node device -config
```

Stop your node process using *ctrl-c*. Check and verify your package.json if it was properly configured.

If the configuration is correct, you can now run your node process using *npm start* command.
```js
$ npm start
```
Your node application should restart automatically after a remote code update, an npm module update, or  a remote restart command is issued from the browser interface.

## Node-M2M Server Query

### Server query to get all available remote devices for each user
```js
const m2m = require('m2m');

const client = new m2m.Client();

client.connect((err, result) => {
  if(err) return console.error('connect error:', err.message);
  console.log('result:', result);

  // server request to get all registered devices
  client.getDevices((err, devices) => {
    if(err) return console.error('getDevices err:', err);
    console.log('devices', devices);
    // devices output
    /*[
      { id: 100, name: 'Machine1', location: 'Seoul, South Korea' },
      { id: 200, name: 'Machine2', location: 'New York, US' },
      { id: 300, name: 'Machine3', location: 'London, UK' }
    ]*/
  });
});
```

### Server query to get each device resources
```js
const m2m = require('m2m');

const client = new m2m.Client();

client.connect((err, result) => {
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  let device = client.accessDevice(300);

  // server request to get device 300 resources
  // e.g. gpio input/output pins, available channels, system information
  device.setupInfo(function(err, data){
    if(err) return console.log('device1 setup error:', err.message);
    console.log('device1 setup data', data);
    // data output
    /*{
      id: 300,
      systemInfo: {
        cpu: 'arm',
        os: 'linux',
        m2mv: '1.2.5',
        totalmem: '4096 MB',
        freemem: '3160 MB'
      },
      gpio: {
        input: { pin: [11, 13], type: 'simulation' },
        output: { pin: [33, 35], type: 'rpi' }
      },
      channel: { name: [ 'voltage', 'gateway1', 'tcp' ] }
    }*/
  });  
});
```
