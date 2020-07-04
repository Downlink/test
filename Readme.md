# m2m

m2m is a client module for machine-to-machine communication framework  [node-m2m](https://www.node-m2m.com).

The module's API is a FaaS (Function as a Service) also called "serverless" without needing to create your own public server infrastructure. Thus, making it easy for everyone to create applications in telematics, telemetry, IoT, data acquisition, network gateways and other machine-to-machine applications.

Each user can create multiple client applications accessing multiple remote devices through its user assigned device/server *id's*. Communications between client and device applications are encrypted using TLS protocol.  Access to remote devices/servers is restricted to authenticated and authorized users only.

[](https://raw.githubusercontent.com/EdoLabs/src/master/m2mSystem2.svg?sanitize=true)



Note:

To use the m2m client module, user must create an account and register their devices with [node-m2m](https://www.node-m2m.com) server.

#### Works with the following devices:

* Raspberry Pi Models: B+, 2, 3, Zero & Zero W, Compute Module 3, 3B+, 3A+ (40-pin models only)
* Linux Computer
* Windows PC
* Mac Computer

## Compatibility

* Node.js Versions: 8.x, 9.x, 10.x, 11.x, 12.x

## Installation
```js
$ npm install m2m
```

This module uses array-gpio for Raspberry Pi peripheral access (GPIO, I2C, SPI and PWM).

You need to install it as a separate module for Raspberry Pi devices.
```js
$ npm install array-gpio
```

## Quick Tour

For this quick tour, we will use two computers communicating with each other using the internet.

One computer will act as the *remote device* that will generate random numbers and simulate a GPIO output using pin 33. It is essentially a basic server providing services such as generating random numbers and simulated gpio output.

The other computer will host a *client application* that will access the random numbers and control the GPIO output using a console.

![](https://raw.githubusercontent.com/EdoLabs/src2/master/quicktour.svg?sanitize=true)
[](quicktour.svg)

Before you start, create an account and register your remote device with node-m2m server.

### Remote Device Setup

Create a device project directory and install m2m module.
```js
$ npm install m2m
```
Create the file below as device.js within your device project directory.
```js
const m2m = require('m2m');

// create a device object with a device id of 100
// this id must must be registered with node-m2m server
let device = new m2m.Device(100);

// your device must be authenticated with m2m server before it starts providing services to clients
device.connect(function(err, result){
    if(err) return console.error('connect error:', err);

    console.log('result:', result);

    // if your credentials are valid, channel 'random' data
    // and simulated gpio output data will be setup

    device.setChannel('random', function(err, data){
      if(err) return console.error('setData random error:', err.message);

      data.result =   Math.floor(( Math.random() * 100) + 25);
      console.log('random value', data.result);
    });

    device.setGpio({type:'simulation', mode:'output', pin:33 }, function (err, gpio){
      if(err) return console.error('output setGpio error:', err.message);

      console.log('gpio output pin', gpio.pin, 'state', gpio.state);
    });
});
```

Start your device application.
```js
$ node device.js or node device.js -r
```

The first time you start your application, you will be prompted to provide your credentials.
```js
$ prompt: Enter userid (email):
$ prompt: Enter password:
$ prompt: Enter security code:

```
Enter your credentials.

The next time you start your application, you don't need to provide your credentials. It will automatically use your saved user token for authentication.

At anytime you can renew your user token using the command line argument **-r** when you start your application.
```js
$ node device.js -r
```
It will ask you to enter your credentials again. Enter your credentials and your token will be renewed.

### Client Application Computer Setup
Similar with the remote device setup, create a client project directory and install m2m module.
```js
$ npm install m2m
```
Create the file below as client.js within your client project directory.
```js
const m2m = require('m2m');

// create a client application object
let client = new m2m.Client();

// enter your credentials for server authentication
client.connect(function(err, result){
    if(err) return console.error('connect error:', err);

    console.log('result:', result);

    // create a device object with access to remote device 100
    let device = client.accessDevice(100);

    // capture 'random' data using a function call
    device.channel('random').getData(function(err, value){
        if(err) return console.error('getData random error:', err.message);
        console.log('random value', value); // 97
    });

    // capture 'random' data using an event-based method
    // data value will be pushed from the remote device if 'random' data value changes
    // remote device will scan/poll the data every 5 secs for any value changes
    device.channel('random').watch(function(err, value){
        if(err) return console.error('watch random error:', err.message);
        console.log('watch random value', value); // 97, 101, 115 ...
    });

    // Turn ON pin 33 of the simulated GPIO output
    device.gpio({mode:'out', pin:33}).on();

    // Turn OFF pin 33 with 2 secs delay
    device.gpio({mode:'out', pin:33}).off(2000);

    // or

    // using an optional callback to confirm the state/status of the simulated GPIO output
    device.gpio({mode:'out', pin:33}).on(function(err, state){
        if(err) return console.error('output pin 33 ON error:', err.message);
	    // true, device is ON
        console.log('remote device output pin 33 ON', state);
    });

    device.gpio({mode:'out', pin:33}).off(2000, function(err, state){
        if(err) return console.error('output pin 33 OFF error:', err.message);
	    // false, device is OFF
        console.log('remote device output pin 33 OFF', state);
    });
});
```
Start your application.
```js
$ node client.js or node client.js -r
```
Similar with remote device setup, you will be prompted to enter your credentials.

You should get an output result as shown below (random values will be differrent) .
```js
...
random value 97
watch random value 81
remote device output pin 33 ON true
remote device output pin 33 OFF false
watch random value 68
watch random value 115
...
```

## Example 1

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

// connect to m2m server for authentication

// explicitly connecting to default m2m server
device.connect('https://www.node-m2m.com', function(err, result){
  // device application logic
});

// or

// implicitly connecting to default m2m server
device.connect(function(err, result){
  if(err) return console.error('connect error:', err);

  console.log('result:', result);

  device.setChannel('temperature', function(err, data){
    if(err) return console.error('set temperature error:', err.message);

    data.result =  i2c.getTemp();
    console.log('temperature value', data.result);
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

  device.channel('temperature').watch(function(err, value){
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

  // watch temperature data by scanning/polling it every 15 secs for any value changes
  device.channel('temperature').watch({interval:15000}, function(err, value){
    if(err) return console.error('temperature error:', err.message);
    console.log('temperature value', value); // 23.51, 23.49, 23.11
  });

  // unwatch temperature data after 5 minutes
  setTimeout(function(){
    device3.channel('temperature').unwatch();
  }, 5*60*1000);
});
```

## Example 2

![](https://raw.githubusercontent.com/EdoLabs/src2/master/example2.svg?sanitize=true)

### Configure device1 for GPIO input monitoring
Install array-gpio both on device1 and device2 for GPIO monitoring and control
```js
$ npm install m2m array-gpio
```

```js
const m2m = require('m2m');

let device = new m2m.Device(120);

device.connect(function(err, result){
  if(err) return console.error('Connect error:', err);

  console.log('result:', result);

  // set pin 11 and 13 as GPIO inputs
  device.setGpio({mode:'input', pin:[11,13]});

  // or using a callback to execute any custom logic

  // callback will be executed if gpio state changes
  device.setGpio({mode:'input', pin:[11,13]}, function(err, gpio){
    if(err) return console.error('input setGpio error:', err.message);

    console.log('input pin', gpio.pin, 'state', gpio.state);
    // process custom logic if callback is executed
  });
});
```

### Configure device2 for GPIO output control
```js
$ npm install m2m array-gpio
```
```js
const m2m = require('m2m');

let device = new m2m.Device(130);

device.connect(function(err, result){
  if(err) return console.error('Connect error:', err);

  console.log('result:', result);

  device.setGpio({mode:'output', pin:[33,35]}, function(err, gpio){
    if(err) return console.error('output setGpio error:', err.message);

    console.log('output pin', gpio.pin, 'state', gpio.state);
  });
});
```

### Client application to control device1 and device2
```js
$ npm install m2m
```

```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(err, result){
    if(err) return console.error('Connect error:', err);
    console.log('result:', result);

    let device1 = client.accessDevice(120);
    let device2 = client.accessDevice(130);

    // using gpio() method for gpio control
    device1.gpio({mode:'in', pin:13}).getState(function(err, state){
      if(err) return console.error('getState pin 13 error:', err.message);
      console.log('device1 input 13 state', state);

      if(state){
        // turn ON device2 output pin 33 immediately
        device2.gpio({mode:'out', pin:33}).on();
        // turn OFF device2 output pin 33 with a time delay of 2 secs
        device2.gpio({mode:'out', pin:33}).off(2000);
      }
    });

    // using input()/in() or output()/out() method for gpio control
    device1.in(11).watch(function(err, state){
      if(err) return console.error('watch pin 11 error:', err.message);
      console.log('device1 input 11 state', state);

      if(state){
        device2.out(35).off();
        return device2.out(33).on(1000);
      }
      device2.out(35).on(function(err, state){
        if(err) return console.error('device2 pin 35 ON state err:', err.message);
        console.log('device2 pin 35 ON state', state);
      });
      device2.out(33).off(1000, function(err, state){
        if(err) return console.error('device2 pin 33 OFF state err:', err.message);
        console.log('device2 pin 33 OFF state', state);
      });
    });
});
```

## Example 3

![](https://raw.githubusercontent.com/EdoLabs/src2/master/example3.svg?sanitize=true)
[](example3.svg)

#### Configure each remote machine's embedded rpi microcontroller with a unique device *id* and set pin 40 for GPIO output control

Install array-gpio on all remote machines for GPIO output control
```js
$ npm install m2m array-gpio
```
```js
const m2m = require('m2m');

// assign 1001, 1002 and 1003 respectively for each remote machine
let device = new m2m.Device(1001);

device.connect((err, result) => {
  if(err) return console.error('Connect error:', err);
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
  if(err) return console.error('Connect error:', err);
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
  let pin = 40; // actuator output pin
  devices.forEach((device, id) => {
    t += 2000;
    device.output(pin).on(t, (err, state) => {
      if(err) return console.error('actuator ON err:', err.message);
      console.log('machine ', id, ' pin ', pin, ' output ON', state);
      device.output(pin).off(t + 400, (err, state) => {
        if(err) return console.error('actuator OFF err:', err.message);
        console.log('machine ', id, ' pin ', pin, ' output OFF', state);
      });
    });
  });
}
```
## Example 4
[](https://raw.githubusercontent.com/EdoLabs/src2/master/example4.svg?sanitize=true)
[](example1.svg)
### Sending Data To Remote Device or Server

#### Client Application
```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(function(err, result){
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  let server = client.accessDevice(500);

  let file = fs.readFileSync('myFile.txt', 'utf8');

  server.channel('send-file').sendData( file , function(err, result){
    if(err) return console.error('send-file error:', err.message);

    console.log('send-file', result); // {send: 'success'}
  });

  let mydata = [{name:'Ed'}, {name:'Jim', age:30}, {name:'Kim', age:42, address:'Seoul, South Korea'}];

  server.channel('send-data').sendData( mydata , function(err, result){
    if(err) return console.error('send-data error:', err.message);

    console.log('send-data', result); // {result: 'success'}
  });

  let num = 1.2456;
  // send a data w/o a response
  server.channel('number').sendData(num);

});
```


#### Device/Server Application
```js
const m2m = require('m2m');

let server = new m2m.Device(500);

server.connect(function(err, result){
  if(err) return console.error('connect error:', err);
  console.log('result:', result);

  server.setChannel('send-file', function(err, data){
    if(err) return console.error('send-file error:', err.message);

    let file = data.payload;

    fs.writeFile('myFile.txt', file, function (err) {
      if (err) throw err;
      console.log('file has been saved!');
      // send back a response
      data.response({send: 'success'});
      // or
      data.json({send: 'success'});
    });
  });

  server.setChannel('send-data', function(err, data){
    if(err) return console.error('send-data error:', err.message);

    console.log('data.payload', data.payload); // [{name:'Ed'}, {name:'Jim', age:30}, {name:'Kim', age:42, address:'Seoul, South Korea'}];
    data.json({result: 'success'});
  });

  server.setChannel('number', function(err, data){
    if(err) return console.error('number error:', err.message);

    console.log('data.payload', data.payload); // 1.2456
  });

});
```


### Http REST API Simulation

#### GET and POST method client request
```js
const m2m = require('m2m');

const client = new m2m.Client();

client.connect((err, result) => {
  if(err) return console.error('Connect error:', err);
  console.log('result:', result);

  // access server 300 using a callback
  client.accessServer(300, (err, server) => {
    if(err) return console.error('accessDevice 300 error:', err.message);

    // GET method api request
    server.api('/random').getData((err, result) => {
      if(err) return console.error('getData random error:', err.message);
      console.log('/random value', result); // 24
    });

    // POST method api request
    server.api('/findData').postData({name:'ed'}, (err, result) => {
      if(err) return console.error('/findData error:', err.message);
      console.log('/findData value', result); // {data: 'ok'}
    });
  });

});
```

#### GET and POST method server setup
```js
const m2m = require('m2m');

const server = new m2m.Server(300);

server.connect((err, result) => {
  if(err) return console.error('Connect error:', err);
  console.log('result:', result);

  // GET method api setup
  server.getApi('/random', (err, data) => {
    if(err) return console.error('/random error:', err.message);
    // set data.value as response
    data.value = Math.floor(( Math.random() * 100) + 25);
    console.log('/random', data.value);
    // or
    data.response(Math.floor(( Math.random() * 100) + 25));
  });

  // POST method api setup
  server.postApi('/findData', (err, data) => {
    if(err) return console.error('/findData error:', err.message);
    setTimeout(() => {
      // received a data.body
      console.log('data.body', data.body); // {name:'ed'}
      // send a response
      data.json({data: 'ok'});
      // or
      data.response({data: 'ok'});
    }, 5000);
  });
});
```

## Browser Interaction

Using the browser, you can download, edit and upload your client/device application code from or into your remote devices anywhere.

To allow the browser to communicate with your application, you need to set a *code* permission option as shown below.
```js
{code:{allow:true, filename:'myApplication.js'}}
```
You need to set the property *allow* to true and provide the *filename* of your application.

Also for client application, you can set a *name*, *location* and *description* property option 
especially if you have multiple clients to monitor as shown below. For device application, you do not need to set these properties.
```js
{code:{allow:true, filename:'myAppFilename.js'}, name:'myAppName', location:'myAppLocation', description:'myAppDescription'}}
```
#### Example

##### Client Application
```js
const m2m = require('m2m');

const client = new m2m.Client();

client.connect(function (err, result) {
  if(err) return console.error('Connect error:', err);
  console.log('result:', result);

  client.setOption({code:{allow:true, filename:'client.js'}, name:'Master App', location:'New York, NY', description:'Main App'});

  // client application logic ...
});
```
##### Device Application
```js
const m2m = require('m2m');

const device = new m2m.Device(1000);

device.connect(function (err, result) {
  if(err) return console.error('Connect error:', err);
  console.log('result:', result);

  // Don't set the name, location and description properties
  // You will provide them during device registration
  device.setOption({code:{allow:true, filename:'device.js'}});

  // device application logic ...
});
```

### Restarting Application or Auto Restart Setup

During browser interaction, your client/device application process will need to be restarted after a module update, application code update, remote command restart etc.

Node-m2m uses nodemon to restart your application process. You can add the following section in your project's package.json as a basic auto restart configuration.
```js
"scripts": {
  "start": "nodemon device.js --delay 2000ms"
},
"nodemonConfig": {
  "verbose": true,
  "restartable": "rs",
  "ignore": [".git", "public"],
  "ignoreRoot": [".git", "public"],
  "execMap": {"js": "node"},
  "watch": ["node_modules/m2m/mon"],
  "env": {"NODE_ENV": "development"},
  "ext": "js,json"
}
```
From the configuration section above, the name of your application is *device.js*. Replace it with the actual name of your application.
 Start your application using npm start.
```js
$ npm start
```
For other custom nodemon configuration, please read nodemon documentation.



## Other FaaS functions

#### Client request to get all available remote devices and resources setup for each device
```js
const m2m = require('m2m');

const client = new m2m.Client();

client.connect((err, result) => {
  if(err) return console.error('Connect error:', err);

  console.log('result:', result);

  // user request to get all registered devices
  client.getDevices((err, devices) => {
	if(err) return console.error('getDevices err:', err.message);
	console.log('devices', devices);
    /*[
      { id: 100, name: 'Machine1', location: 'Seoul, South Korea' },
      { id: 200, name: 'Machine2', location: 'New York, US' },
      { id: 300, name: 'Machine3', location: 'London, UK' }
    ]*/
  });

  // user request to get device 300 resources
  client.accessDevice(300, (err, device) => {
    if(err) return console.error('accessDevice 300 error:', err.message);
    // e.g. gpio input/output pins, available channels, system information
    device.setupInfo(function(err, data){
      if(err) return console.log('device1 setup error:', err.message);
      console.log('device1 setup data', data);
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
