---
title: Avimesa Gadget
date: 2020-05-21
slug: avimesa-gadget

---
Avimesa Gadget is a lightweight software tool that provides Linux-based devices, like a Raspberry Pi, with data transmission capabilities into the Avimesa Industrial IoT Solution. Using your favorite programming language to drive Gadget, you can transfer digital sensor readings into the cloud where they are then made available to the web endpoint of your choice.

## Installation

1.  [Create an Avimesa.Live Account][1] if you haven't already.
2.  [Download Avimesa Gadget][2] onto a Raspberry Pi and unzip the .deb file.
3.  Run `sudo dpkg -i <file-name>.deb`

## Description

Avimesa Gadget operates as a virtual Avimesa device and provides data ingestion and cloud communication functionalities. Gadget input data is formatted using JSON (JavaScript Object Notation) and adheres to the Avimesa Dialtone protocol. Accepted input data is stored for the duration of Gadget execution or until overwritten and transmitted when commanded to do so. Gadget offers various modes of operation: an interactive command-line interface (CLI), control via STDIN, and single message transmission. For a more in-depth description, please see Gadget's man page by typing `man avmsagadget` in the terminal, or see 'Full Usage' below.

## Simple Usage

1.  (via CLI) Run `avmsagadget -i <device-id> <auth-code>` If you don't have a "device-id" and its associated "auth-code" or you don't know what these are, read about them here.
2.  Now you're using "interactive" Gadget! Let's try the `info` command: this displays all of Gadget's internal settings.
3.  Let's see if Gadget has any data (it shouldn't right now): run `info CHANNELS`
4.  Inject some data: `{"ch_idx":0,"ch_data":[{"data_idx":0,"units":1,"val":5.0}]}` Again, this command may be foreign to you, please read more about what this is here.
5.  Let's check for data again: `info CHANNELS`
6.  Finally, run the command `run` This will attempt to send data through the Avimesa system and, with your Avimesa.Live account, you should be able to see it on the frontend!

## Full Usage

### Name

avmsagadget - data transmission interface into the Avimesa Industrial IoT Solution

### Synopsis

avmsagadget [options] auth-key

### Description

avmsagadget, or simply Gadget, operates as a virtual Avimesa device and provides data ingestion and cloud communication functionalities. Gadget input data is formatted using JSON (JavaScript Object Notation) and adheres to the Avimesa Dialtone protocol. For more information on the Avimesa Dialtone protocol, please visit www.avimesa.com.

Accepted input data is stored for the duration of Gadget execution or until overwritten and transmitted when commanded to do so.

Gadget offers various modes of operation: an interactive CLI (tty), control via STDIN, and single message transmission. All of Gadget's forms of execution will fail if an auth-key is not provided; more information on the auth-key input follows.

Single message transmission (-j option): input data is passed in at avmsagadget invocation, transmission is attempted once, and finally termination occurs.

Controlling Gadget via STDIN and Gadget running as an interactive CLI execute in a similar persistent fashion and require some driving. Both persistent modes of operation maintain an internal state and are controlled and manipulated by a simple set of commands. Gadget's internal state during both persistent modes of operation consists of ingested data as well as a few settings that manipulate execution. IMPORTANT: commands piped into Gadget during persistent operation are delimited by new lines ('\n').

Both persistent modes of operation are terminated whenever an EOF is received via STDIN.

A note on avmsagadget and the -p option: if -p is specified, three user-determined named pipes are created in the filesystem for Gadget's STDIN, STDOUT, and STDERR.

For users new to the Avimesa Industrial IoT solution, a brief introduction to the Avimesa Dialtone protocol is most likely necessary. An Avimesa-enabled device, be it hardware or software, must have both a valid Device ID and a valid Authentication Key (auth-key) for cloud communication. For information on obtaining a valid set of these 32 character hexadecimal values (UUIDs), please visit www.avimesa.com.

Data in the world of Avimesa (Dialtone introduction continuation) is stored in channels. An Avimesa-enabled device can have up to 64 channels and each channel can containup to sixteen 32 bit data elements. Each data element can have one of the following types: boolean, integer, floating-point, or string. Following is an example of a JSON-formatted channel Dialtone with floating-point data:

```
 {
     "chans": [
     {
         "ch_idx": 0,
         "ch_data": [
         {
             "data_idx": 0,
             "units", 1,
             "val", 7.2
         },
         ...
         ]
     },
     ...
     ]
 }
 ```
    

### Options

**-i device-id** A 32-byte hexadecimal Device ID Gadget should execute with.

**-j json** (single transmission mode) Causes Gadget to attempt to send JSON-formatted channel Dialtone (json) to the cloud.

**-p path-to-file** Execute Gadget with its STDIN, STDOUT, and STDERR existing as named pipes in the filesystem at path-to-file.

### Settings

**DEVICEID** Gadget's 32 byte Device ID (defaults to 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF if not read from configuration file).

**AUTHCODE** Gadget's 32-byte Authentication Key.

**HOSTNAME** The hostname of the web endpoint Gadget should attempt to communicate with (defaults to "devices.avimesacorp.net" if not read from the configuration file).

**PORTNUM** The port of the web endpoint Gadget should attempt to communicate with (defaults to 36360 if not read from the configuration file).

**AUTORUN** When set, causes the multi-channel form of the { command to automatically initiate a cloud data transmission (defaults to 0).

**COMMANDS** The following commands are available in interactive CLI operation as well as in control via STDIN.

set setting [=] value Attempts to set setting to value

**{json-input}** Attempts to input JSON-formatted channel Dialtone (json-input) into Gadget. This command offers two forms of operation: single-channel and multi-channel. When operating in single-channel mode, the user will attempt to update a single channel within Gadget. Input data for single-channel mode must take the form of a single channel within the channel array of the JSON-formatted channel Dialtone object (starting with "ch_idx"). When operating in multi-channel mode, the input is expected to have the complete JSON-formatted channel Dialtone object ("chans"). Accepted input via multi-channel mode will overwrite all existing channel data. IMPORTANT: commands piped into Gadget during persistent operation are delimited by newlines ('\n') (don't inject any until the end of your JSON input).

**run**  
Attempts to communicate with the Avimesa cloud using Gadget's current internal state.

**exit**  
Terminates Gadget.

**info [channels]** Prints Gadget's internal settings. If run with channels, will print Gadget's current ingested channels.

**help** Prints helpful information.

### Files

/etc/avimesa.conf.d/avmsagadget.conf (configuration file) Can be used to (mostly) initialize Gadget. This file should contain three lines: (1) Device ID, (2) hostname, and (3) port number.

Avimesa Gadget is available as a [free download][2].

 [1]: /sign-up_tutorial/
 [2]: /downloads/