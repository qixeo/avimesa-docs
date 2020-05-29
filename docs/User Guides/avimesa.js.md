---
title: Avimesa.js
date: 2020-05-28
slug: avimesa-js

---
This guide will walk you through the process of installing Avimesa.js using a shell script, which automatically installs Avimesa.js and its dependencies. A common development scenario would be to run Avimesa Gadget on a Raspberry Pi and the web application (Avimesa Sails) on localhost with Mac or Windows until it's ready to deploy. For a detailed primer on the Avimesa Sails application [please see this document][1].

#### Avimesa.js consists of several products:

*   Avimesa.js Web App (Avimesa Sails)
*   Avimesa Gadget
*   Avimesa 1000 Simulator

#### Pre-requisites for this installation:

*   Machine running Debian Linux or ARM Linux** (Avimesa Gadget currently requires one of these) ** *A Raspberry Pi is a great choice*
*   Zip file containing Avimesa.js [Download here][2]
*   [Avimesa.Live account][3] with your API credentials handy

#### Instructions:

If you haven't already created your free Avimesa.Live Account [click here][3]

Open a terminal window and change into the directory where Avimesa.js-1.0.zip is and run the command `unzip Avimesa.js-1.0.zip`

Type `cd Avimesa.js-1.0` and `enter`

Run `sudo ./install.sh`

*   It may take anywhere from 5-15 minutes to install depending on your internet connection 
*   This will install NodeJS, SailsJS, Avimesa Gadget and run npm install in order to install all required node_modules to run Avimesa Sails

Run `./changeCredentials.sh`

*   Enter your `Avimesa Api Key` and `Avimesa Api Password`

Type `cd avimesa-sailsjs` then `enter`

Run the following command to launch the Avimesa Sails application:
```
    sails lift
```

Now, open a second Terminal window

Change directory back to `Avimesa.js-1.0` by typing `cd ../`

Run this command `cd Avimesa\ 1000\ Simulator`

Then run the following command to launch the Avimesa 1000 Simulator"
```
    node index.js
```  

Open two browser windows and navigate to one of the following:

*   on SSH to Pi - `your_ip_address:8080` in one window and `your_ip_address:1337` in another, or:
*   on Debian/Desktop - `localhost:8080` in one window and `localhost:1337` in the other

You should now have two browser windows open. The window running port 1337 should look like this, running Avimesa Sails:

<img src="http://13.52.82.105/wp-content/uploads/2019/09/Avimesa-Sails-Login-1024x598.jpg" alt="" width="1024" height="598" class="aligncenter size-large wp-image-3944 shadow-small rounded-corners" />

And the window running port 8080 should look like this, running the Avimesa 1000 Simulator:

<img src="http://13.52.82.105/wp-content/uploads/2019/09/avimesa-1000-simulator-1024x598.jpg" alt="" width="1024" height="598" class="aligncenter size-large wp-image-3945 shadow-small rounded-corners" />

In the *Avimesa Sails* window (port 1337), login with:

*   Username: `admin@example.com`

*   Password: `abc123`

Go to `Devices` and click on the `Device ID` with which you want to send data from

Click on `Add Sensor` on top right of sensors info

Add a new Amp (AC) sensor on `channel 1` with min reading `0` and max `5`

Add a new Temp (Fahrenheit) sensor on `channel 2` with min reading `0` and max `300`

On the *Avimesa 1000 Simulator* web page (port 8080), Enter `Device ID` and `Authcode` for that device and how many messages per DELAY seconds you want to send and click `Run`

Click on a sensor that you added to view a chart of the sensor data you sent. You should now see a chart with data that looks something like this:

<img src="https://avimesa.com/wp-content/uploads/2019/08/IoT-data-chart-temperature-1024x624.png" alt="IoT temperature data chart" width="1024" height="624" class="aligncenter size-large wp-image-3626 shadow-small rounded-corners" />

Note: You may install the apps individually without the Avimesa.js installer script. For instructions see the following:

[Installing Avimesa Sails][1]

[Installing Avimesa Gadget][4]

 [1]: https://avimesa.com/docs/user-guides/avimesa-sails/
 [2]: https://avimesa.com/downloads/
 [3]: https://avimesa.com/create-account/
 [4]: /avimesa-gadget/