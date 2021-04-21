# m2m

m2m is a client module for machine-to-machine communication framework  [node-m2m](https://www.node-m2m.com).

The module's API is a FaaS (Function as a Service) also called "serverless" making it easy for everyone to develop applications in telematics, data acquisition, process automation, network gateways, workflow orchestration and many others.

Provision multiple public device servers on the fly from anywhere. Clients will access the device servers through its user assigned device *id*.   

Access to devices is restricted to authenticated users only. Communications between client and device/server applications are fully encrypted using TLS protocol.

To use this module, users must create an account and register their devices with [node-m2m](https://www.node-m2m.com).

[](https://raw.githubusercontent.com/EdoLabs/src/master/m2mSystem2.svg?sanitize=true)

# Table of contents
1. [Supported Devices](#supported-devices)
2. [Node.js version requirement](#nodejs-version-requirement)
3. [Installation](#installation)
4. [Quick Tour](#quick-tour)
5. Examples
   - [Using MCP 9808 Temperature Sensor](#example-1-using-mcp-9808-temperature-sensor)
   - [GPIO Input Monitor and Output Control](#example-2-gpio-input-monitor-and-output-control)
   - [Remote Machine Control](#example-3-remote-machine-control)
   - [Sending Data to Remote Device](#example-4-sending-data-to-remote-device)
6. HTTP API
    - [Server GET and POST method](#server-get-and-post-method)
    - [Client GET and POST request](#client-get-and-post-request)
7. Using The Browser Interface
   - [Naming Your Client Application for Tracking Purposes](#naming-your-client-application-for-tracking-purposes)
   - [Remote Application Code Editing](#remote-application-code-editing)
   - [Auto Restart Setup](#auto-restart-setup)
   - [Auto Configuration for Code Edit and Auto Restart](#auto-configuration-for-code-edit-and-auto-restart)
8. Node-M2M Server Query
   - [Server query to get all available remote devices](#server-query-to-get-all-available-remote-devices)
   - [Server query to get each device resources](#server-query-to-get-each-device-resources)


## Supported Devices

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

For this quick tour, we will let two computers communicate with each other using the internet.

We will create a server (*remote device*) that will generate random numbers as its sole service.

And a client application (*remote client*) that will access the random numbers.

We will access the random numbers in two methods.

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

// authenticate and connect with node-m2m server
device.connect(function(err, result){
    if(err) return console.error('connect error:', err);

    console.log('result:', result);

    // set the random data using a random generator  
    device.setData('random', function(err, data){
      if(err) return console.error('setData random error:', err.message);

      let rvvalue =   Math.floor(( Math.random() * 100) + 25);
      data.send(value);
    });
});
```

Start your device application.
```js
$ node device.js
```

The first time you run your application, it will ask for your credentials.
```js
? Enter your userid (email):
? Enter your password:
? Enter your security code:

```
The next time you run your application, it will start automatically without asking for your credentials. It will use a saved user token for authentication.

At anytime you can renew your user token using the command line argument **-r** when you restart your application.
```js
$ node device.js -r
```
It will ask you to enter your credentials again. Enter your credentials to renew your token.

### Remote Client Setup
Similar with the remote device setup, create a client project directory and install m2m.
```js
$ npm install m2m
```
Create one the file below as client.js within your client project directory.

There are two ways clients can access the data from the remote device.

1. Create a local device object with access to the remote device. This method is convenient if we need only to access one remote device server. You provide only once the device id of the device server you want to access.
```js
const m2m = require('m2m');

// create a client application object
let client = new m2m.Client();

client.connect(function(err, result){
    if(err) return console.error('connect error:', err);

    console.log('result:', result);

    // create a local device object with access to remote device 100
    let device = client.accessDevice(100);

    // capture 'random' data using a one-time function call
    device.getData('random', function(err, value){
        if(err) return console.error('getData random error:', err.message);
        console.log('random value', value); // 97
    });

    // capture 'random' data using an event-based method
    // the remote device will scan/poll the data every 5 secs (default) for any value changes
    // if the value changes, it will be pushed/sent to the client
    device.watch('random', function(err, value){
        if(err) return console.error('watch random error:', err.message);
        console.log('watch random value', value); // 81, 68, 115 ...
    });
});
```
2. Access the remote device directly from the client object and provide the device id every time for each request method. This is handy if we have multiple remote device servers to access;    
```js
const m2m = require('m2m');

// create a client application object
let client = new m2m.Client();

client.connect(function(err, result){
    if(err) return console.error('connect error:', err);

    console.log('result:', result);

    // get 'random' data using a one-time function call
    client.getData(100, 'random', function(err, value){
        if(err) return console.error('getData random error:', err.message);
        console.log('random value', value); // 97
    });

    // watch 'random' data using an event-based method
    client.watch(100, 'random', function(err, value){
        if(err) return console.error('watch random error:', err.message);
        console.log('watch random value', value); // 81, 68, 115 ...
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
...
random value 97
watch random value 81
watch random value 68
watch random value 115
...
```

## Examples

### Example 1 Using MCP 9808 Temperature Sensor

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

    let value =  i2c.getTemp();
    data.send(value);
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

  device.watch('temperature', function(err, value){
    if(err) return console.error('temperature error:', err.message);
    console.log('temperature value', value); // 23.51, 23.49, 23.11
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

  // scan/poll the data every 15 secs
  // instead of the default 5 secs
  //device.watch({channel:'temperature', interval:15000}, function(err, value){
  device.watch('temperature', 15000}, function(err, value){
    if(err) return console.error('temperature error:', err.message);
    console.log('temperature value', value); // 23.51, 23.49, 23.11
  });

  // unwatch temperature data after 5 minutes
  // client will stop receiving temperature data from the remote device
  setTimeout(function(){
    device.unwatch('temperature');
  }, 5*60000);
});
```

### Example 2 GPIO Input Monitor and Output Control

![](https://raw.githubusercontent.com/EdoLabs/src2/master/example2.svg?sanitize=true)

### Configure device1 for gpio input monitoring
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

  device.setGpio({mode:'input', pin:[11, 13]}, function(err, gpio){
    if(err) return console.error('setGpio input error:', err.message);

    console.log('input pin', gpio.pin, 'state', gpio.state);
    // add your custom logic here
  });
});
```

### Configure device2 for gpio output control
```js
$ npm install m2m array-gpio
```
```js
const m2m = require('m2m');

let device = new m2m.Device(130);

device.connect(function(err, result){
  if(err) return console.error('connect error:', err);

  console.log('result:', result);

  device.setGpio({mode:'output', pin:[33, 35]}, function(err, gpio){
    if(err) return console.error('setGpio output error:', err.message);

    console.log('output pin', gpio.pin, 'state', gpio.state);
    // add your custom logic here
  });
});
```
### Client to monitor device1 gpio inputs and control device2 gpio outputs
```js
$ npm install m2m
```
There are two ways we can access the gpio input and output pins from our remote devices.

The first one is to create a local device object for each remote devices.
```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(err, result){
    if(err) return console.error('connect error:', err);
    console.log('result:', result);

    // create a local device object for each device
    let device1 = client.accessDevice(120);
    let device2 = client.accessDevice(130);

    // using gpio() method for gpio input pin monitoring
    // and output pin control
    device1.gpio({mode:'in', pin:11}).watch(function(err, state){
      if(err) return console.error('watch pin 13 error:', err.message);
      console.log('device1 input 11 state', state);

      if(state){
        console.log('turn ON device2 output 33');
        device2.gpio({mode:'out', pin:33}).on();
      }
      else{
        console.log('turn OFF device2 output 33');
        device2.gpio({mode:'out', pin:33}).off();
      }
    });

    // using in/input() or out/output() method
    // for gpio input pin monitoring and output pin control
    device1.in(13).watch(function(err, state){
      if(err) return console.error('watch pin 11 error:', err.message);
      console.log('device1 input 13 state', state);

      if(state){
        console.log('turn OFF device2 output 35');
        device2.out(35).off();
      }
      else{
        console.log('turn ON device2 output 35');
        device2.out(35).on();
      }
    });
});
```
The second one is to access the input and output pins directly from the client object by providing the device id of each remote device.
```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(err, result){
    if(err) return console.error('connect error:', err);
    console.log('result:', result);
    // watch pin 11 from device 120
    client.input(120, 11).watch(function(err, state){    
      if(err) return console.error('watch pin 11 error:', err.message);
      console.log('device 120 input 11 state', state);

      if(state){
        console.log('turn ON device 130 output 33');
        client.output(130, 33).on();
      }
      else{
        console.log('turn OFF device 130 output 33');
        client.output(130, 33).off();
      }
    });
    // watch pin 13 from device 120
    client.input(120, 13).watch(function(err, state){
      if(err) return console.error('watch pin 13 error:', err.message);
      console.log('device 120 input 13 state', state);

      if(state){
        console.log('turn OFF device 130 output 35');
        client.output(130, 35).off();
      }
      else{
        console.log('turn ON device 120 output 35');
        client.output(130, 35).on();
      }
    });
});
```

### Example 3 Remote Machine Control

![](https://raw.githubusercontent.com/EdoLabs/src2/master/example3.svg?sanitize=true)
[](example3.svg)

#### Configure each remote machine's embedded rpi microcontroller with a unique device *id* <br> and set pin 40 for GPIO output control

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
  // set pin 40 as digital output to activate the actuator
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

  // accessing multiple devices using an array object
  let devices = client.accessDevice([1001, 1002, 1003]);

  clearInterval(interval);
  machineControl(devices);
  // iterate over the remote machines every 15 secs
  // turn on each actuator one by one every 2 seconds
  // then, turning them off likewise
  interval = setInterval(machineControl, 15000, devices);

  // or

  client.accessDevice([1001, 1002, 1003], (err, devices) => {
    if(err) return console.error('accessDevice error:', err.message);

    clearInterval(interval);
    machineControl(devices);
    interval = setInterval(machineControl, 15000, devices);
  });
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
### Example 4 Sending Data to Remote Device
[](https://raw.githubusercontent.com/EdoLabs/src2/master/example4.svg?sanitize=true)
[](example1.svg)

#### Device/Server
```js
const m2m = require('m2m');
const fs = require('fs');

let server = new m2m.Device(500);

server.connect(function(err, result){
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  // channel 'send-file' service
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

  // channel 'send-data' service
  server.setData('send-data', function(err, data){
    if(err) return console.error('send-data error:', err.message);

    // data.payload  [{name:'Ed'}, {name:'Jim', age:30}, {name:'Kim', age:42, address:'Seoul, South Korea'}];
    console.log('data.payload', data.payload);
    // send a response
    if(Array.isArray(data.payload)){
      data.send({data: 'valid'});
    }
    else{
      data.send({data: 'invalid'});
    }
  });

  // channel 'number' service
  server.setData('number', function(err, data){
    if(err) return console.error('number error:', err.message);

    console.log('data.payload', data.payload); // 1.2456
  });

});
```

#### Client
```js
const fs = require('fs');
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(err, result){
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  let server = client.accessDevice(500);

  // sending a text file to a remote server
  let myfile = fs.readFileSync('myFile.txt', 'utf8');

  server.sendData('send-file', myfile , function(err, result){
    if(err) return console.error('send-file error:', err.message);

    console.log('send-file', result); // {result: 'success'}
  });

  // sending json data to a remote server
  let mydata = [{name:'Ed'}, {name:'Jim', age:30}, {name:'Kim', age:42, address:'Seoul, South Korea'}];

  server.sendData('send-data', mydata , function(err, result){
    if(err) return console.error('send-data error:', err.message);

    console.log('send-data', result); // {data: 'valid'}
  });

  // sending data to a remote server w/o a response
  let num = 1.2456;
  server.sendData('number', num);

});
```

### HTTP API

#### Server GET and POST method
```js
const m2m = require('m2m');

const server = new m2m.Server(300);

server.connect((err, result) => {
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  // GET method api setup
  server.getApi('/random', (err, data) => {
    if(err) return console.error('/random error:', err.message);

    data.send(Math.floor(( Math.random() * 100) + 25));
  });

  // POST method api setup
  server.postApi('/findData', (err, data) => {
    if(err) return console.error('/findData error:', err.message);
    setTimeout(() => {
      // received a data.body
      console.log('data.body', data.body); // {name:'ed'}
      // send a response
      data.send({result: 'ok'});
    }, 5000);
  });
});
```

#### Client GET and POST request
```js
const m2m = require('m2m');

const client = new m2m.Client();

client.connect((err, result) => {
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  // access server 300 using a callback
  client.accessServer(300, (err, server) => {
    if(err) return console.error('accessDevice 300 error:', err.message);

    // GET method api request
    //server.api('/random').get((err, result) => {
    server.get('random/data', (err, data) => {    
      if(err) return console.error('/random error:', err.message);
      console.log('get random', data); // 24
    });

    // POST method api request
    //server.api('/findData').post({name:'ed'}, (err, result) => {
    server.post('random/data', {name:'ed'} , (err, data) => {   
      if(err) return console.error('/findData error:', err.message);
      console.log('post random/data', data); // {result: 'ok'}
    });
  });

});
```

## Using The Browser Interface

### Naming Your Client Application for Tracking Purposes

Unlike *device/server* applications, users can create *client* applications on the fly without needing to register it with **node-m2m** server.

Node-m2m tracks all client applications through a dynamic *client id*.
If you have multiple client applications, it may be difficult to track all your clients by just referring to its *client id* from the browser interface.

You can assign a **name**, **location** and a **description** properties when you create a client object as shown below.

This makes it easy to track all your clients using the browser interface by referring to its name, location and description instead of just looking at the client id's which are basically just random alpha-numeric id's.
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

Using the browser interface, you can download, edit and upload your application code from or into your remote clients and devices from anywhere.

To allow the browser to communicate with your application, you need to edit your project's package.json and add the following *m2mConfig* property
as shown below.


```js
"m2mConfig": {
  "code": {
    "allow": true,
    "filename": "device.js"
  }
}
```
You need to set the property *allow* to true and provide the *filename* of your application.

From the example above, the filename of the application is *device.js*. Replace it with the actual filename of your application.


### Auto Restart Setup

Using the browser interface, you may need to restart your application after a module update, application code edit/update, remote restart command etc.

Node-m2m uses **nodemon** to restart your application.

You can add the following *nodemonConfig* and *scripts* properties in your project's npm package.json as a basic auto restart configuration.
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

## Auto Configuration for Code Edit and Auto Restart
To automatically configure your package.json for code editing and auto restart, start your node process with -config flag.

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

Your node process or application will automatically restart after a remote code update, an npm module update, a remote restart command etc. using the browser interface.


## Node-M2M Server Query

### Server query to get all available remote devices
```js
const m2m = require('m2m');

const client = new m2m.Client();

client.connect((err, result) => {
  if(err) return console.error('connect error:', err.message);
  console.log('result:', result);

  // user request to get all registered devices
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

  client.accessDevice(300, (err, device) => {
    if(err) return console.error('accessDevice 300 error:', err.message);

    // client request to get device 300 resources
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
});
```
