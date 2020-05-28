---
title: Avimesa Group API (AMQP)
date: 2020-05-28
slug: avimesa-group-api-amqp

---
*last updated 2019-Aug-22*

## Introduction

This project contains the Avimesa Group API documentation. The purpose of this documentation is to describe the API available for the Avimesa Group clients. The Group API (AMQP) in general can be seen as a JSON based protocol that uses Avimesa-configured RabbitMQ for messaging and provides the ability to interface with the Avimesa system.

<a id="toc"></a>

## Table of Contents

*   [1\. Summary][1] 
    *   [1\.1 Overview][2]
    *   [1\.2 Client Examples][3]
    *   [1\.3 References][4]
    *   [1\.4 Terminology][5]
*   [2\. Components][6] 
    *   [2\.1 Avimesa Core Architecture][7] 
        *   [2\.1.1 Devices][8]
        *   [2\.1.2 Avimesa Messages Servers][9]
        *   [2\.1.3 Script Engine][10]
        *   [2\.1.4 Group vhosts][11]
        *   [2\.1.5 Group vhost Client][12]
*   [3\. Recommended Client Usage][13] 
    *   [3\.1 Consumer/User/Producer][14]
    *   [3\.2 Data Consumer Worker][15]
    *   [3\.3 Device Actuation Process][16]
    *   [3\.4 Syslog Consumer Worker][17]
    *   [3\.5 Admin Worker Process][18]
*   [4\. Group API][19] 
    *   [4\.1 General Data Flow][20] 
        *   [4\.1.1 AMQP Permissions][21]
        *   [4\.1.2 Command Requests][22]
        *   [4\.1.3 Command Responses][23]
        *   [4\.1.4 Raw Data Subscription][24]
        *   [4\.1.5 Notification Data Subscription][25]
        *   [4\.1.6 System Log Data Subscription][26]
    *   [4\.2 List Devices - Command 1002][27]
    *   [4\.3 Add a Device - Command 1004][28]
    *   [4\.4 List a Device's Files - Command 1006][29]
    *   [4\.5 Upload a Device Driver Script File - Command 1008][30]
    *   [4\.6 Upload a Device Configuration File - Command 1010][31]
    *   [4\.7 Remove a Device - Command 1012][32]
    *   [4\.8 Update a Device's Authentication Key - Command 1014][33]
    *   [4\.9 Upload a Device Firmware Update Package - Command 1020][34]
    *   [4\.10 Actuation][35] 
        *   [4\.10.1 Summary][36]
        *   [4\.10.2 Set GPIO State - Command 0xB102][37]
        *   [4\.10.3 Soft Reset - Command 0xF002][38]
        *   [4\.10.4 Firmware Update Trigger - Command 0xF008][39]
    *   [4\.11 System Log][40] 
        *   [4\.11.1 Summary][41]
        *   [4\.11.2 JSON Format][42]
        *   [4\.11.3 System Log Event IDs][43]
*   [5\. DialTone QuickStart][44] 
    *   [5\.1 Summary][45]
    *   [5\.2 Typical Device Data][46]
    *   [5\.3 Typical Device Configuration][47]

The following lists larger API updates. See github for complete changelog of the smaller changes.

| API Version | Change                                                                                                                                      |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| 0\.10       | Initial release on github                                                                                                                   |
| 0\.11       | - Changed CMD 1XXX responses. Instead of a generic "message" arrary of strings, the response is now a JSON object that is easier to consume |

<a id="1.-summary"></a>

## 1\. Summary

<a id="1.1-summary"></a>

### 1\.1 Overview

The Group API supports the following:

**Direct Device Interaction** - Ability to consume/subscribe to device data (raw/notification) - Queue based data access - Ability to actuate devices (i.e. send commands that elicit a device action)

**Device Management and Provisioning** - Ability provision and remove devices - Ability to set a device script and configuration - Ability to reset device authentication keys - Ability to list devices/device files

**Device and System Administration** - update device firmware (support for Avimesa 1000, but in general provides agnostic DFU style operations) - view system logs

**Security** - TLS1.2 - Group Level Authentication (128-bit ID/128-bit Auth Key)

<a id="1.2-summary"></a>

### 1\.2 Client Examples

Client examples showing various use cases are available in source code form are located here:

*   Node.js SDK [npm package][48] and source code on [GitHub][49]
*   Node.js Examples using SDK on [GitHub][50]

<a id="1.3-references"></a>

### 1\.3 References

| Document                      | Summary                                                                                                                                                                                       |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Avimesa_RMQ_Design.pdf        | Detailed information about the RabbitMQ configuration and interface, target audience being a developer. Can be used in conjunction with this document when building an Infrastructure client. |
| Avimesa_DialTone_Protocol.pdf | Detailed information about the DialTone protocol                                                                                                                                              |

*Table 1*

<a id="1.4-terminology"></a>

### 1\.4 Terminology

| Term                        | Meaning                                                                                                                                                                                       |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Group                       | A term that is typically used to represent a single customer, it’s a logical grouping of Devices for an end user, and can be represented with a single Group ID                               |
| Device                      | Typically a system in the field that communicates with the Avimesa Messages.                                                                                                                  |
| vhost                       | Virtual Host – a data isolation mechanism that allows Groups to be isolated from each other (i.e. customer X can’t access customer Y’s data). Other uses include an Infrastructure interface. |
| Core Architecture           | In this scope, represents a collection of hardware and software that facilitates Device to Avimesa Messages communications and messaging.                                                     |
| Infrastructure Architecture | In this scope, represents software systems that facilitate account creation, maintenance and accounting bookkeeping of Device to Avimesa Messages transactions.                               |

*Table 2*

[Top][51]  
<a id="2.-components"></a>

## 2\. Components

<a id="2.1-components"></a>

### 2\.1 Avimesa Core Architecture

In general, the Avimesa architecture can be viewed as follows for the scope of this document.

![fig1][52]  
*Figure 1*

<a id="2.1.1-components"></a>

#### 2\.1.1 Devices

In the scope of this document, a device is something that provides data that the Group wants to know about, is uniquely identified and authenticated against, and is configured via the Group vhost. The Device’s data will make its way through the system and will be available to the client application.

<a id="2.1.2-components"></a>

#### 2\.1.2 Avimesa Messages Servers

In the scope of this document, the Avimesa Messages server manage all device communications, device authentication and hosting of the device runtimes.

<a id="2.1.3-components"></a>

#### 2\.1.3 Script Engine

A JavaScript program is executed on each device transaction (connection) by a Device Driver Script Engine. It provides the ability to send data to the Group vhosts data queues. In the scope of this document, it should be known that if the script does not manipulate the data models, than the default Avimesa DialTone formatted JSON can be expected in the data queues.

The script can also pull device actuations out of the Device’s queue and relay to the device. In general, the script provides:

*   a JavaScript runtime for business logic
*   Access to the device’s data/state
*   Access to write to data queues (raw/notification)
*   Access to Read from device’s actuation queue
*   Some file I/O for hysteresis

The script allows for the data models to be manipulated and thus can be considered a data adapter.

In general, it’s easy to think of this script as a filtering and priority setting mechanism for data.

<a id="2.1.4-components"></a>

#### 2\.1.4 Group vhosts

A “Group vhost” is created per Group (customer). This vhost provides the end user access to their device data, device configuration and administration. In the context of this document, a Group vhost should be considered the bridge between a customer application and their devices. The customer’s application is responsible for connecting to the Group vhost using the various RMQ clients available (or writing their own). Avimesa has examples in various languages. The credentials to connect to the Group vhost are configured at the time of Group creation, which as of now is an Avimesa internal service.

<a id="2.1.5-components"></a>

#### 2\.1.5 Group vhost Client

An application that interfaces with the Group vhost needs to use (or build) a RabbitMQ client using AMQP 0-9-1. The thought is this document provides the necessary information to assist in designing a Group client.

[Top][51]  
<a id="3.-client-usage"></a>

## 3\. Recommended Client Usage

The following is provided to give an overall sense of the Group API’s usage. When the Avimesa Infrastructure API is used to create a new Group (Customer), the Group’s vhost is created, the end user is given credentials, and the API is available for use.

[Top][51]  
<a id="3.1-client-usage"></a>

### 3\.1 Example

Complete examples of a simple client usage are located here:

*   Node.js SDK [npm package][48] and source code on [GitHub][49]
*   Node.js Examples using SDK on [GitHub][50]

[Top][51]  
<a id="3.2-client-usage"></a>

### 3\.2 Data Consumer Worker

![fig3][53]  
*Figure 3*

*   The Avimesa queues should be considered ‘temporary’ storage and ‘offloaded’ on a periodic basis to a data store
*   A worker should subscribe or consume data from the raw and notification data queues The data in the raw and notification queue can be populated via the Device’s script through the JavaScript API
*   The format of the JSON data will be Avimesa’s DialTone unless it is re-formatted via the Device’s script, in which is can be anything the user desires (in JSON)

[Top][51]  
<a id="3.3-client-usage"></a>

### 3\.3 Device Actuation Process

![fig4][54]  
*Figure 4*

*   A Device Actuation Process should be considered as something that can generate commands for specific devices based upon the DialTone JSON protocol
*   Device’s can be ‘actuated’ (commands sent to have an action take place on the device) by having an application send the JSON command to the Device’s queue via the actuation exchange
*   The Device may obtain the JSON command via the Device’s script through the JavaScript

[Top][51]  
<a id="3.4-client-usage"></a>

### 3\.4 Syslog Consumer Worker

![fig5][55]  
*Figure 5*

*   The Avimesa queues should be considered ‘temporary’ storage and ‘offloaded’ on a periodic basis to a data store
*   A worker should subscribe or consume data from the system log queues
*   The information in this queue should be viewed as system log type information, error conditions, etc.
*   The format of the JSON data will be Avimesa’s DialTone

[Top][51]  
<a id="3.5-client-usage"></a>

### 3\.5 Admin Worker Process

#### Summary

![fig6][56]  
*Figure 6*

Most of the Group API can be seen as a client sending 'admin' commands to the system, and processing the response to these commands. In general, messages are sent to the `admin_in_q` queue via the `admin.dx` exchange with a routing key of `in`. The client can check for the response in two ways, described below.

#### Remote Procedure Call (RPC)

The RPC approach is clean and allows for clients to easily get responses only to the commands it sends. This is useful for 'multi-tenant' use cases.

In general, this is the usage model, with a source code example available [here][57], please see the `sendAdminCmd` function.

*   a connection is created (channel)
*   create a random `correlationId` (request ID) (I use a random 32 bit number in string form)
*   create a new temporary queue; nameless, exclusive, 60 second expiration, autodelete
*   begin to consume on that new queue (subscribe)
*   publish the message to `admin.dx` exchange, routing key = `in`, set options on the message to have the `replyTo` property set to the temporary queue name and `correlationID` to ID created above. The temporary queue will have a name like "amq.gen-"
*   Once you receive the response in the temporary queue, close the connection, and the temporary queue should go away.

**Note:** the `admin_out_q` is not used at all in this usage model.

#### Single Consumer Usage

It may suffice to have a single consumer of all admin command responses.

*   An Admin Worker process is responsible for both generating and sending admin commands following the DialTone JSON format and processing the responses via a response in a queue
*   This function is asynchronous in nature and should use the DialTone’s request ID to process the response
*   JSON requests are sent to the admin exchange (`admin.dx`) with a routing key (`in`) following the DialTone JSON format to perform actions on the system
*   The responses to the requests are available to be subscribed to or consumed from the admin out queue (`admin_out_q`)

[Top][51]  
<a id="4.-group-api"></a>

## 4\. Group API

[Top][51]  
<a id="4.1-group-api"></a>

### 4\.1 General Data Flow

<a id="4.1.1-group-api"></a>

#### 4\.1.1 AMQP Permissions

The default Group API permissions are as follows, which allow resources to be configured if they follow the 'nameless' pattern, and full read/write access.

`"amq.gen-.*" ".*" ".*"`

For the RabbitMQ specifics, please reference Avimesa_RMQ_Design.pdf.

<a id="4.1.2-group-api"></a>

#### 4\.1.2 Command Requests (Admin In)

Requests are sent to a RMQ Exchange via the following RMQ settings:

<table>
  <thead>
    <tr>
      <th>
      </th>
      
      <th>
      </th>
    </tr>
  </thead>
  
  <tbody>
    <tr>
      <td>
        <strong>vhost</strong>
      </td>
      
      <td>
        Provided to user
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Host</strong>
      </td>
      
      <td>
        rmqserv001.avimesa.com
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Port</strong>
      </td>
      
      <td>
        5671
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Credentials</strong>
      </td>
      
      <td>
        Provided to user
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Exchange</strong>
      </td>
      
      <td>
        Name = “admin.dx”, Type = Direct, Passive = True
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Routing Key</strong>
      </td>
      
      <td>
        “in” (for admin in)
      </td>
    </tr>
  </tbody>
</table>

<a id="4.1.3-group-api"></a>

#### 4\.1.3 Command Response (Admin Out)

Responses are available by default via the following RMQ settings. It should be noted that a nice RPC style approach can be used instead where a temporary response queue is created and used for the commands response. See

<table>
  <thead>
    <tr>
      <th>
      </th>
      
      <th>
      </th>
    </tr>
  </thead>
  
  <tbody>
    <tr>
      <td>
        <strong>vhost</strong>
      </td>
      
      <td>
        Provided to user
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Host</strong>
      </td>
      
      <td>
        rmqserv001.avimesa.com
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Port</strong>
      </td>
      
      <td>
        5671
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Credentials</strong>
      </td>
      
      <td>
        Provided to user
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Queue</strong>
      </td>
      
      <td>
        Name = <code>admin_out_q</code>, Passive = True, Durable = True
      </td>
    </tr>
  </tbody>
</table>

<a id="4.1.4-group-api"></a>

#### 4\.1.4 Raw Data Subscription

Raw data records are published and can be subscribed to. The consumer should acknowledge the message once consumed.

<table>
  <thead>
    <tr>
      <th>
      </th>
      
      <th>
      </th>
    </tr>
  </thead>
  
  <tbody>
    <tr>
      <td>
        <strong>vhost</strong>
      </td>
      
      <td>
        Provided to user
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Host</strong>
      </td>
      
      <td>
        rmqserv001.avimesa.com
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Port</strong>
      </td>
      
      <td>
        5671
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Credentials</strong>
      </td>
      
      <td>
        Provided to user
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Queue</strong>
      </td>
      
      <td>
        Name = <code>raw_q</code>, Passive = True, Durable = True
      </td>
    </tr>
  </tbody>
</table>

<a id="4.1.5-group-api"></a>

#### 4\.1.5 Notification Data Subscription

Notification data records are published and can be subscribed to. The consumer should acknowledge the message once consumed.

<table>
  <thead>
    <tr>
      <th>
      </th>
      
      <th>
      </th>
    </tr>
  </thead>
  
  <tbody>
    <tr>
      <td>
        <strong>vhost</strong>
      </td>
      
      <td>
        Provided to user
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Host</strong>
      </td>
      
      <td>
        rmqserv001.avimesa.com
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Port</strong>
      </td>
      
      <td>
        5671
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Credentials</strong>
      </td>
      
      <td>
        Provided to user
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Queue</strong>
      </td>
      
      <td>
        Name = <code>not_q</code>, Passive = True, Durable = True
      </td>
    </tr>
  </tbody>
</table>

<a id="4.1.6-group-api"></a>

#### 4\.1.6 System Log Data Subscription

System log data records are published and can be subscribed to. The consumer should acknowledge the message once consumed.

<table>
  <thead>
    <tr>
      <th>
      </th>
      
      <th>
      </th>
    </tr>
  </thead>
  
  <tbody>
    <tr>
      <td>
        <strong>vhost</strong>
      </td>
      
      <td>
        Provided to user
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Host</strong>
      </td>
      
      <td>
        rmqserv001.avimesa.com
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Port</strong>
      </td>
      
      <td>
        5671
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Credentials</strong>
      </td>
      
      <td>
        Provided to user
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Queue</strong>
      </td>
      
      <td>
        Name = <code>sys_log_q</code>, Passive = True, Durable = True
      </td>
    </tr>
  </tbody>
</table>

[Top][51]  
<a id="4.2-group-api"></a>

### 4\.2 List Devices - Command 1002

**Summary**

Requests to list the Devices in the Group.

**Request Example (1002)**

    {
       "api_maj": 0,
       "api_min": 11,
       "cmd_id": 1002,
       "req_id": 123456
    }
    

| Name    | Type           | Required | Notes                                                   |
| ------- | -------------- | -------- | ------------------------------------------------------- |
| api_maj | Number, uint16 | Yes      | API major version (provided by Avimesa)                 |
| api_min | Number, uint16 | Yes      | API minor version (provided by Avimesa)                 |
| cmd_id  | Number, uint32 | Yes      | Command ID                                              |
| req_id  | Number, uint32 | Yes      | Client generated Request ID to track back to a response |

**Response Example (1003)**

    {
        "api_maj": 0, 
        "api_min": 11,
        "cmd_id": 1003,
        "req_id": 123456,
        "response": {
            "error": 0,
            "message": {
                devices : [
                    "20010db80000000053d371fffe53dc33",
                    "20010db80000000053d371fffe53dc34",
                    "20010db80000000053d371fffe53dc35" 
                ]
            }
        } 
    }
    

| Name     | Type           | Required | Notes                                                          |
| -------- | -------------- | -------- | -------------------------------------------------------------- |
| api_maj  | Number, uint16 | Yes      | API major version (provided by Avimesa)                        |
| api_min  | Number, uint16 | Yes      | API minor version (provided by Avimesa)                        |
| cmd_id   | Number, uint32 | Yes      | Command ID                                                     |
| req_id   | Number, uint32 | Yes      | Echoed client generated Request ID to track back to a response |
| response | Object         | Yes      | (see next section)                                             |

| Name    | Type           | Required | Notes                             |
| ------- | -------------- | -------- | --------------------------------- |
| error   | Number, uint32 | Yes      | 0 for success, non-zero for error |
| message | Object         | Yes      | (see next section)                |

| Name    | Type   | Required | Notes                           |
| ------- | ------ | -------- | ------------------------------- |
| devices | Arrary | Yes      | An array of “Device ID” strings |

[Top][51]  
<a id="4.3-group-api"></a>

### 4\.3 Add a Device - Command 1004

**Summary**

Requests to add a Device to the Group.

**Request Example (1004)**

    {
        "api_maj": 0,
        "api_min": 11,
        "cmd_id": 1004,
        "req_id": 123456,
        "dev_id": "20010db800000000f7706ffffe1e34c6"
    }
    

| Name    | Type                                | Required | Notes                                                   |
| ------- | ----------------------------------- | -------- | ------------------------------------------------------- |
| api_maj | Number, uint16                      | Yes      | API major version (provided by Avimesa)                 |
| api_min | Number, uint16                      | Yes      | API minor version (provided by Avimesa)                 |
| cmd_id  | Number, uint32                      | Yes      | Command ID                                              |
| req_id  | Number, uint32                      | Yes      | Client generated Request ID to track back to a response |
| dev_id  | String, 32 char lowercase, 0-9, a-f | Yes      | The Device ID to add                                    |

**Response Example (1005)**

    {
        "api_maj": 0, 
        "api_min": 11, 
        "cmd_id": 1003, 
        "req_id": 123456, 
        "response": {
            "error": 0,
            "message": {
                "device_id" : "20010db800000000f7706ffffe1e34c6"
                "auth_key" : "2ff87d3a0d064d278ecff756aa57ebf6"
            }
        }
    }
    

| Name     | Type           | Required | Notes                                                          |
| -------- | -------------- | -------- | -------------------------------------------------------------- |
| api_maj  | Number, uint16 | Yes      | API major version (provided by Avimesa)                        |
| api_min  | Number, uint16 | Yes      | API minor version (provided by Avimesa)                        |
| cmd_id   | Number, uint32 | Yes      | Command ID                                                     |
| req_id   | Number, uint32 | Yes      | Echoed client generated Request ID to track back to a response |
| response | Object         | Yes      | (see next section)                                             |

| Name    | Type           | Required | Notes                             |
| ------- | -------------- | -------- | --------------------------------- |
| error   | Number, uint32 | Yes      | 0 for success, non-zero for error |
| message | Object         | Yes      | (see next section)                |

| Name     | Type                                | Required | Notes                  |
| -------- | ----------------------------------- | -------- | ---------------------- |
| dev_id   | String, 32 char lowercase, 0-9, a-f | Yes      | The Device ID to added |
| auth_key | String                              | Yes      | The Authentication Key |

[Top][51]  
<a id="4.4-group-api"></a>

### 4\.4 List a Device's Files - Command 1006

**Summary**

Requests to list a Device’s files.

**Request Example (1006)**

    {
        "api_maj": 0,
        "api_min": 11,
        "cmd_id": 1006,
        "req_id": 123456,
        "dev_id": "20010db800000000f7706ffffe1e34c6"
    }
    

| Name    | Type                                | Required | Notes                                                   |
| ------- | ----------------------------------- | -------- | ------------------------------------------------------- |
| api_maj | Number, uint16                      | Yes      | API major version (provided by Avimesa)                 |
| api_min | Number, uint16                      | Yes      | API minor version (provided by Avimesa)                 |
| cmd_id  | Number, uint32                      | Yes      | Command ID                                              |
| req_id  | Number, uint32                      | Yes      | Client generated Request ID to track back to a response |
| dev_id  | String, 32 char lowercase, 0-9, a-f | Yes      | The Device ID to add                                    |

**Response Example (1007)**

    {
        "api_maj": 0, 
        "api_min": 11, 
        "cmd_id": 1007, 
        "req_id": 123456,
        "response": {
            "error": 0, 
            "message": {
                "files" : [
                    {
                        "path" : "/scripts/script.js",
                        "size" : 590,
                        "time" : 1536350169
                    },
                    {
                        "path" : "/config/config.js",
                        "size" : 1169,
                        "time" : 1536350193
                    }
                ]
            }
        }
    }
    

| Name     | Type           | Required | Notes                                                          |
| -------- | -------------- | -------- | -------------------------------------------------------------- |
| api_maj  | Number, uint16 | Yes      | API major version (provided by Avimesa)                        |
| api_min  | Number, uint16 | Yes      | API minor version (provided by Avimesa)                        |
| cmd_id   | Number, uint32 | Yes      | Command ID                                                     |
| req_id   | Number, uint32 | Yes      | Echoed client generated Request ID to track back to a response |
| response | Object         | Yes      | (see next section)                                             |

| Name    | Type           | Required | Notes                             |
| ------- | -------------- | -------- | --------------------------------- |
| error   | Number, uint32 | Yes      | 0 for success, non-zero for error |
| message | Object         | Yes      | (see next section)                |

| Name  | Type  | Required | Notes                                                                                            |
| ----- | ----- | -------- | ------------------------------------------------------------------------------------------------ |
| files | Array | Yes      | Will contain an array of “file” strings with the path, size (bytes) and upload time (Linux time) |

[Top][51]  
<a id="4.5-group-api"></a>

### 4\.5 Upload a Device Driver Script File - Command 1008

**Summary**

Requests to upload a Device Drive Script file.

**Request Example (1008)**

    {
        "api_maj": 0,
        "api_min": 11,
        "cmd_id": 1008,
        "req_id": 123456,
        "dev_id": "20010db800000000f7706ffffe1e34c6",
        “file_buf”: "ewogICJkZXZpY2VzIiA6IDEwLAogICJtb2RlIiA6IDEsCiAgInJ
            tcV9pcF9hZGRyZXNzIiA6ICIyMDcuMzYuNDYuMjEiLAogICJybXFfcG9ydCIgOiA1 NjcyLAogICJybXFfdmhvc3QiIDogIlRlc3RHcm91cCIsCiAgInJtcV91c2VyIiA6ICJhdmltZXNhX2 FkbWluIiwKICAicm1xX3Bhc3N3b3JkIiA6ICJaV 1V5TXpFNVpEUmxaVFV3TVRsbVptUTRaR0U1WlRWaiIKfQ=="
    }
    

| Name     | Type                                | Required | Notes                                                   |
| -------- | ----------------------------------- | -------- | ------------------------------------------------------- |
| api_maj  | Number, uint16                      | Yes      | API major version (provided by Avimesa)                 |
| api_min  | Number, uint16                      | Yes      | API minor version (provided by Avimesa)                 |
| cmd_id   | Number, uint32                      | Yes      | Command ID                                              |
| req_id   | Number, uint32                      | Yes      | Client generated Request ID to track back to a response |
| dev_id   | String, 32 char lowercase, 0-9, a-f | Yes      | The Device ID                                           |
| file_buf | String                              | Yes      | Base64 encoded version of the file contents             |

**Response Example (1009)**

    {
        "api_maj": 0,
        "api_min": 11,
        "cmd_id": 1009,
        "req_id": 123456,
        "response": {
            "error": 0, 
        } 
    }
    

| Name     | Type           | Required | Notes                                                          |
| -------- | -------------- | -------- | -------------------------------------------------------------- |
| api_maj  | Number, uint16 | Yes      | API major version (provided by Avimesa)                        |
| api_min  | Number, uint16 | Yes      | API minor version (provided by Avimesa)                        |
| cmd_id   | Number, uint32 | Yes      | Command ID                                                     |
| req_id   | Number, uint32 | Yes      | Echoed client generated Request ID to track back to a response |
| response | Object         | Yes      | (see next section)                                             |

| Name    | Type           | Required | Notes                             |
| ------- | -------------- | -------- | --------------------------------- |
| error   | Number, uint32 | Yes      | 0 for success, non-zero for error |
| message | Object         | No       | (see next section)                |

| Name   | Type   | Required | Notes                                    |
| ------ | ------ | -------- | ---------------------------------------- |
| status | String | Yes      | Contains status message or error message |

[Top][51]  
<a id="4.6-group-api"></a>

### 4\.6 Upload a Device Configuration File - Command 1010

**Summary**

**Request Example (1010)**

    {
        "api_maj": 0,
        "api_min": 11,
        "cmd_id": 1010,
        "req_id": 123456,
        "dev_id": "20010db800000000f7706ffffe1e34c6",
        “file_buf”: "ewogICJkZXZpY2VzIiA6IDEwLAogICJtb2RlIiA6IDEsCiAgInJ
            tcV9pcF9hZGRyZXNzIiA6ICIyMDcuMzYuNDYuMjEiLAogICJybXFfcG9ydCIgOiA1 NjcyLAogICJybXFfdmhvc3QiIDogIlRlc3RHcm91cCIsCiAgInJtcV91c2VyIiA6ICJhdmltZXNhX2 FkbWluIiwKICAicm1xX3Bhc3N3b3JkIiA6ICJaV 1V5TXpFNVpEUmxaVFV3TVRsbVptUTRaR0U1WlRWaiIKfQ=="
    }
    

| Name     | Type                                | Required | Notes                                                   |
| -------- | ----------------------------------- | -------- | ------------------------------------------------------- |
| api_maj  | Number, uint16                      | Yes      | API major version (provided by Avimesa)                 |
| api_min  | Number, uint16                      | Yes      | API minor version (provided by Avimesa)                 |
| cmd_id   | Number, uint32                      | Yes      | Command ID                                              |
| req_id   | Number, uint32                      | Yes      | Client generated Request ID to track back to a response |
| dev_id   | String, 32 char lowercase, 0-9, a-f | Yes      | The Device ID                                           |
| file_buf | String                              | Yes      | Base64 encoded version of the file contents             |

**Response Example (1011)**

    {
        "api_maj": 0,
        "api_min": 11,
        "cmd_id": 1011,
        "req_id": 123456,
        "response": {
            "error": 0, 
        } 
    }
    

| Name     | Type           | Required | Notes                                                          |
| -------- | -------------- | -------- | -------------------------------------------------------------- |
| api_maj  | Number, uint16 | Yes      | API major version (provided by Avimesa)                        |
| api_min  | Number, uint16 | Yes      | API minor version (provided by Avimesa)                        |
| cmd_id   | Number, uint32 | Yes      | Command ID                                                     |
| req_id   | Number, uint32 | Yes      | Echoed client generated Request ID to track back to a response |
| response | Object         | Yes      | (see next section)                                             |

| Name    | Type           | Required | Notes                             |
| ------- | -------------- | -------- | --------------------------------- |
| error   | Number, uint32 | Yes      | 0 for success, non-zero for error |
| message | Object         | No       | (see next section)                |

| Name   | Type   | Required | Notes                                    |
| ------ | ------ | -------- | ---------------------------------------- |
| status | String | Yes      | Contains status message or error message |

[Top][51]  
<a id="4.7-group-api"></a>

### 4\.7 Remove a Device - Command 1012

**Summary**

Requests to add a Remove to the Device (and its container/files/access to the Avimesa Messages)

**Request Example (1012)**

    {
        "api_maj": 0,
        "api_min": 11,
        "cmd_id": 1012,
        "req_id": 123456,
        "dev_id": "20010db800000000f7706ffffe1e34c6"
    }
    

| Name    | Type                                | Required | Notes                                                   |
| ------- | ----------------------------------- | -------- | ------------------------------------------------------- |
| api_maj | Number, uint16                      | Yes      | API major version (provided by Avimesa)                 |
| api_min | Number, uint16                      | Yes      | API minor version (provided by Avimesa)                 |
| cmd_id  | Number, uint32                      | Yes      | Command ID                                              |
| req_id  | Number, uint32                      | Yes      | Client generated Request ID to track back to a response |
| dev_id  | String, 32 char lowercase, 0-9, a-f | Yes      | The Device ID to remove                                 |

**Response Example (1013)**

    {
        "api_maj": 0,
        "api_min": 11,
        "cmd_id": 1011,
        "req_id": 123456,
        "response": {
            "error": 0, 
        } 
    }
    

| Name     | Type           | Required | Notes                                                          |
| -------- | -------------- | -------- | -------------------------------------------------------------- |
| api_maj  | Number, uint16 | Yes      | API major version (provided by Avimesa)                        |
| api_min  | Number, uint16 | Yes      | API minor version (provided by Avimesa)                        |
| cmd_id   | Number, uint32 | Yes      | Command ID                                                     |
| req_id   | Number, uint32 | Yes      | Echoed client generated Request ID to track back to a response |
| response | Object         | Yes      | (see next section)                                             |

| Name    | Type           | Required | Notes                             |
| ------- | -------------- | -------- | --------------------------------- |
| error   | Number, uint32 | Yes      | 0 for success, non-zero for error |
| message | Object         | No       | (see next section)                |

| Name   | Type   | Required | Notes                                    |
| ------ | ------ | -------- | ---------------------------------------- |
| status | String | Yes      | Contains status message or error message |

[Top][51]  
<a id="4.8-group-api"></a>

### 4\.8 Update a Device's Authentication Key - Command 1014

**Summary**

Requests to reset to update Device authentication key. **NOTE** for devices like the Avimesa 1000, you'll need to update the authentication key on the device in order for it to communicate again with the Avimesa Messages.

**Request Example (1014)**

    {
        "api_maj": 0,
        "api_min": 11,
        "cmd_id": 1014,
        "req_id": 123456,
        "dev_id": "20010db800000000f7706ffffe1e34c6"
    }
    

| Name    | Type                                | Required | Notes                                                   |
| ------- | ----------------------------------- | -------- | ------------------------------------------------------- |
| api_maj | Number, uint16                      | Yes      | API major version (provided by Avimesa)                 |
| api_min | Number, uint16                      | Yes      | API minor version (provided by Avimesa)                 |
| cmd_id  | Number, uint32                      | Yes      | Command ID                                              |
| req_id  | Number, uint32                      | Yes      | Client generated Request ID to track back to a response |
| dev_id  | String, 32 char lowercase, 0-9, a-f | Yes      | The Device ID to update                                 |

**Response Example (1015)**

    {
        "api_maj": 0,
        "api_min": 11,
        "cmd_id": 1015,
        "req_id": 123456,
        "response": {
            "error": 0,
            "message": {
                "device_id" : "20010db800000000f7706ffffe1e34c6"
                "auth_key" : "2ff87d3a0d064d278ecff756aa57ebf6"
            }
        }
    }
    

| Name     | Type           | Required | Notes                                                          |
| -------- | -------------- | -------- | -------------------------------------------------------------- |
| api_maj  | Number, uint16 | Yes      | API major version (provided by Avimesa)                        |
| api_min  | Number, uint16 | Yes      | API minor version (provided by Avimesa)                        |
| cmd_id   | Number, uint32 | Yes      | Command ID                                                     |
| req_id   | Number, uint32 | Yes      | Echoed client generated Request ID to track back to a response |
| response | Object         | Yes      | (see next section)                                             |

| Name    | Type           | Required | Notes                             |
| ------- | -------------- | -------- | --------------------------------- |
| error   | Number, uint32 | Yes      | 0 for success, non-zero for error |
| message | Object         | Yes      | (see next section)                |

| Name     | Type                                | Required | Notes                  |
| -------- | ----------------------------------- | -------- | ---------------------- |
| dev_id   | String, 32 char lowercase, 0-9, a-f | Yes      | The Device ID updated  |
| auth_key | String                              | Yes      | The Authentication Key |

[Top][51]  
<a id="4.9-group-api"></a>

### 4\.9 Upload a Device Firmware Update Package - Command 1020

**Summary**

Requests to upload a Device Firmware Update Pakcage file.

Currently supports:

*   Avimesa 1000 (nRF52382)

**Request Example (1020)**

    {
        "api_maj": 0,
        "api_min": 11,
        "cmd_id": 1020,
        "req_id": 123456,
        "dev_id": "20010db800000000f7706ffffe1e34c6",
        “file_buf”: "ewogICJkZXZpY2VzIiA6IDEwLAogICJtb2RlIiA6IDEsCiAgInJ
            tcV9pcF9hZGRyZXNzIiA6ICIyMDcuMzYuNDYuMjEiLAogICJybXFfcG9ydCIgOiA1 NjcyLAogICJybXFfdmhvc3QiIDogIlRlc3RHcm91cCIsCiAgInJtcV91c2VyIiA6ICJhdmltZXNhX2 FkbWluIiwKICAicm1xX3Bhc3N3b3JkIiA6ICJaV 1V5TXpFNVpEUmxaVFV3TVRsbVptUTRaR0U1WlRWaiIKfQ=="
    }
    

| Name     | Type                                | Required | Notes                                                   |
| -------- | ----------------------------------- | -------- | ------------------------------------------------------- |
| api_maj  | Number, uint16                      | Yes      | API major version (provided by Avimesa)                 |
| api_min  | Number, uint16                      | Yes      | API minor version (provided by Avimesa)                 |
| cmd_id   | Number, uint32                      | Yes      | Command ID                                              |
| req_id   | Number, uint32                      | Yes      | Client generated Request ID to track back to a response |
| dev_id   | String, 32 char lowercase, 0-9, a-f | Yes      | The Device ID                                           |
| file_buf | String                              | Yes      | Base64 encoded version of the file contents             |

**Response Example (1021)**

    {
        "api_maj": 0,
        "api_min": 11,
        "cmd_id": 1021,
        "req_id": 123456,
        "response": {
            "error": 0, 
        }
    }
    

| Name     | Type           | Required | Notes                                                          |
| -------- | -------------- | -------- | -------------------------------------------------------------- |
| api_maj  | Number, uint16 | Yes      | API major version (provided by Avimesa)                        |
| api_min  | Number, uint16 | Yes      | API minor version (provided by Avimesa)                        |
| cmd_id   | Number, uint32 | Yes      | Command ID                                                     |
| req_id   | Number, uint32 | Yes      | Echoed client generated Request ID to track back to a response |
| response | Object         | Yes      | (see next section)                                             |

| Name    | Type           | Required | Notes                             |
| ------- | -------------- | -------- | --------------------------------- |
| error   | Number, uint32 | Yes      | 0 for success, non-zero for error |
| message | Object         | No       | (see next section)                |

| Name   | Type   | Required | Notes                                    |
| ------ | ------ | -------- | ---------------------------------------- |
| status | String | Yes      | Contains status message or error message |

[Top][51]  
<a id="4.10-group-api"></a>

### 4\.10 Actuation

[Top][51]  
<a id="4.10.1-group-api"></a>

#### 4\.10.1 Summary

The command ID's shift from the 1000's series to a hexadecimal flavor. This helps distinguishing Group API commands with Device commands.

Requests are sent to a RMQ Exchange via the following RMQ settings:

<table>
  <thead>
    <tr>
      <th>
      </th>
      
      <th>
      </th>
    </tr>
  </thead>
  
  <tbody>
    <tr>
      <td>
        <strong>vhost</strong>
      </td>
      
      <td>
        Provided to user
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Host</strong>
      </td>
      
      <td>
        rmqserv001.avimesa.com
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Port</strong>
      </td>
      
      <td>
        5671
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Credentials</strong>
      </td>
      
      <td>
        Provided to user
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Exchange</strong>
      </td>
      
      <td>
        Name = “actuation.dx”, Type = Direct, Passive = True
      </td>
    </tr>
    
    <tr>
      <td>
        <strong>Routing Key</strong>
      </td>
      
      <td>
        The Device’s ID, 32 character, <strong>lower case</strong>
      </td>
    </tr>
  </tbody>
</table>

Responses are provided by the device **but must be forwarded by the script to a queue if you would like to act upon an ACK**

[Top][51]  
<a id="4.10.2-group-api"></a>

#### 4\.10.2 Set GPIO State - Command 0xB102

**Summary**

Requests to set GPIO state of the device

**Request Example (0xB102)**

    {
        "api_maj":0,
        "api_min":11,
        "dts":1539090421,
        "dev": {
            "dev_id":"000102030405060708090A0B0C0D0E0F", 
            "dev_cmd":{
                "dev_cmd_id":45314,
                “req_id”:12345,
                "state":1,
                "mask":1
            } 
        }
    }
    

| Name    | Type           | Required | Notes                                             |
| ------- | -------------- | -------- | ------------------------------------------------- |
| api_maj | Number, uint16 | Yes      | API major version (provided by Avimesa)           |
| api_min | Number, uint16 | Yes      | API minor version (provided by Avimesa)           |
| dts     | Number, uint32 | Yes      | Linux timestamp (used for filtering, can be 0...) |
| dev     | Object         | Yes      | See next section                                  |

| Name    | Type   | Required | Notes                                     |
| ------- | ------ | -------- | ----------------------------------------- |
| dev_id  | String | Yes      | The Device’s ID, 32 character, lower case |
| dev_cmd | Object | Yes      | See next section                          |

| Name       | Type           | Required | Notes                                                 |
| ---------- | -------------- | -------- | ----------------------------------------------------- |
| dev_cmd_id | Number, uint32 | Yes      | he Command ID, represented in base 10                 |
| req_id     | Number, uint32 | Yes      | The Request ID, used to track request to the response |
| state      | Number, uint32 | Yes      | See below                                             |
| mask       | Number, uint32 | Yes      | See below                                             |

<a id="4.10.2.1-group-api"></a>

##### 4\.10.2.1 State

*   Supports up to 32 pins in the 32 bit value
*   When bit is 0 GPIO is off, when bit is 1 GPIO is on
*   For example, 0x00000003 would enable GPIO 0 and 1

<a id="4.10.2.2-group-api"></a>

##### 4\.10.2.2 Mask

*   Bit wise flags, 0 – ignore, 1 – don’t ignore.
*   For example, a mask of 0x00000001 on state of 0x00000003 Flags above would inform device to enable. GPIO 0 but not GPIO 1. This allows us to prevent disabling GPIO (e.g. request doesn’t need to have system state)

**Response Example (0xB103)**

**NOTE: Device Driver Script needs to relay the ACK to a queue**

    {
        "api_maj" : 0,
        "api_min" : 11,
        "dts" : 0,
        "dev" : {
            "dev_cmd" : {
                "dev_cmd_id" : 45315,
                "req_id" : 123456, 
                }
        } 
    }
    

| Name    | Type           | Required | Notes                                             |
| ------- | -------------- | -------- | ------------------------------------------------- |
| api_maj | Number, uint16 | Yes      | API major version (provided by Avimesa)           |
| api_min | Number, uint16 | Yes      | API minor version (provided by Avimesa)           |
| dts     | Number, uint32 | Yes      | Linux timestamp (used for filtering, can be 0...) |
| dev     | Object         | Yes      | See next section                                  |

| Name    | Type   | Required | Notes            |
| ------- | ------ | -------- | ---------------- |
| dev_cmd | Object | Yes      | See next section |

| Name       | Type           | Required | Notes                                                 |
| ---------- | -------------- | -------- | ----------------------------------------------------- |
| dev_cmd_id | Number, uint32 | Yes      | The Command ID, represented in base 10                |
| req_id     | Number, uint32 | Yes      | The Request ID, used to track request to the response |

[Top][51]  
<a id="4.10.3-group-api"></a>

#### 4\.10.3 Soft Reset - Command 0xF002

**Summary**

Requests to a soft reset of device

**Request Example**

    {
        "api_maj" : 0,
        "api_min" : 11,
        "dts" : 0,
        "dev" : {
            "dev_id": “cafebabecafebabecafebabecafebabe”,
            "dev_cmd" : {
                "dev_cmd_id" : 61442,
                "req_id" : 123456 
            }
        }
    }
    

| Name    | Type           | Required | Notes                                             |
| ------- | -------------- | -------- | ------------------------------------------------- |
| api_maj | Number, uint16 | Yes      | API major version (provided by Avimesa)           |
| api_min | Number, uint16 | Yes      | API minor version (provided by Avimesa)           |
| dts     | Number, uint32 | Yes      | Linux timestamp (used for filtering, can be 0...) |
| dev     | Object         | Yes      | See next section                                  |

| Name    | Type   | Required | Notes                                     |
| ------- | ------ | -------- | ----------------------------------------- |
| dev_id  | String | Yes      | The Device’s ID, 32 character, lower case |
| dev_cmd | Object | Yes      | See next section                          |

| Name       | Type           | Required | Notes                                                 |
| ---------- | -------------- | -------- | ----------------------------------------------------- |
| dev_cmd_id | Number, uint32 | Yes      | The Command ID, represented in base 10                |
| req_id     | Number, uint32 | Yes      | The Request ID, used to track request to the response |

**Response Example**

NONE IN CURRENT API

[Top][51]

<a id="4.10.4-group-api"></a>

#### 4\.10.4 Firmware Update Trigger - Command 0xF008

**Summary**

Requests to start Device Firmware Update process

**Request Example (0xF008)**

    {
        "api_maj" : 0,
        "api_min" : 11,
        "dts" : 0,
        "dev" : {
            "dev_id": “cafebabecafebabecafebabecafebabe”, 
            "dev_cmd" : {
                "dev_cmd_id" : 61448,
                "req_id" : 123456
            }
        } 
    }
    

| Name    | Type           | Required | Notes                                             |
| ------- | -------------- | -------- | ------------------------------------------------- |
| api_maj | Number, uint16 | Yes      | API major version (provided by Avimesa)           |
| api_min | Number, uint16 | Yes      | API minor version (provided by Avimesa)           |
| dts     | Number, uint32 | Yes      | Linux timestamp (used for filtering, can be 0...) |
| dev     | Object         | Yes      | See next section                                  |

| Name    | Type   | Required | Notes                                     |
| ------- | ------ | -------- | ----------------------------------------- |
| dev_id  | String | Yes      | The Device’s ID, 32 character, lower case |
| dev_cmd | Object | Yes      | See next section                          |

| Name       | Type           | Required | Notes                                                 |
| ---------- | -------------- | -------- | ----------------------------------------------------- |
| dev_cmd_id | Number, uint32 | Yes      | The Command ID, represented in base 10                |
| req_id     | Number, uint32 | Yes      | The Request ID, used to track request to the response |

**Response Example (0xF009)**

**NOTE: Device Driver Script needs to relay the ACK to a queue**

    {
        "api_maj" : 0,
        "api_min" : 11,
        "dts" : 0,
        "dev" : {
            "dev_cmd" : {
                "dev_cmd_id" : 61449,
                "req_id" : 123456, 
                }
        } 
    }
    

| Name    | Type           | Required | Notes                                             |
| ------- | -------------- | -------- | ------------------------------------------------- |
| api_maj | Number, uint16 | Yes      | API major version (provided by Avimesa)           |
| api_min | Number, uint16 | Yes      | API minor version (provided by Avimesa)           |
| dts     | Number, uint32 | Yes      | Linux timestamp (used for filtering, can be 0...) |
| dev     | Object         | Yes      | See next section                                  |

| Name    | Type   | Required | Notes            |
| ------- | ------ | -------- | ---------------- |
| dev_cmd | Object | Yes      | See next section |

| Name       | Type           | Required | Notes                                                 |
| ---------- | -------------- | -------- | ----------------------------------------------------- |
| dev_cmd_id | Number, uint32 | Yes      | The Command ID, represented in base 10                |
| req_id     | Number, uint32 | Yes      | The Request ID, used to track request to the response |

[Top][51]  
<a id="4.11-system-log"></a>

### 4\.11 System Log

<a id="4.11.1-system-log"></a>

#### 4\.11.1 Summary

The System Log is accessed through the `sys_loq_q` queue. It's very similar to the Raw (`raw_q`) and Notification (`not_q`) queues, but will generally contain system events and errors in JSON form.

<a id="4.11.2-system-log"></a>

#### 4\.11.2 JSON Format

    {
        “evt_id” : 65539, 
        “dts” : 1539383229, 
        “dev_id” : “0e8155d8559543860000000000000001", 
        “msg” : “Encoding dev_out to JSON failed”}
    }
    

| Name   | Type           | Required | Notes                                         |
| ------ | -------------- | -------- | --------------------------------------------- |
| evt_id | Number, uint32 | Yes      | The event ID (see below)                      |
| dts    | Number, uint32 | Yes      | Linux time of event                           |
| dev_id | String         | No       | The device ID that created the event (if any) |
| msg    | String         | Yes      | Additional text for the message               |

<a id="4.11.3-system-log"></a>

#### 4\.11.3 System Log Event IDs

##### API Events Overview - 0x00010000 (65536)

| Event ID (base 16/10) | Origin     | Notes                            |
| --------------------- | ---------- | -------------------------------- |
| 0x00010000 (65537)    | API (AMQP) | General API Event                |
| 0x00011000 (69632)    | API (AMQP) | General Infrastructure API Event |
| 0x00012000 (73728)    | API (AMQP) | General Group API Event          |

##### Group API Events - 0x00012000 (73728)

| Event ID (base 16/10) | Origin     | Notes                   |
| --------------------- | ---------- | ----------------------- |
| 0x00012000 (73728)    | API (AMQP) | General Group API Event |
| 0x00012001 (73729)    | API (AMQP) | Unknown Command Error   |
| 0x00012002 (73730)    | API (AMQP) | Command Parsing Error   |
| 0x00012010 (73744)    | API (AMQP) | CMD 1000 - Success      |
| 0x00012011 (73745)    | API (AMQP) | CMD 1000 - Error        |
| 0x00012012 (73746)    | API (AMQP) | CMD 1002 - Success      |
| 0x00012013 (73747)    | API (AMQP) | CMD 1002 - Error        |
| 0x00012014 (73748)    | API (AMQP) | CMD 1004 - Success      |
| 0x00012015 (73749)    | API (AMQP) | CMD 1004 - Error        |
| 0x00012016 (73750)    | API (AMQP) | CMD 1006 - Success      |
| 0x00012017 (73751)    | API (AMQP) | CMD 1006 - Error        |
| 0x00012018 (73752)    | API (AMQP) | CMD 1008 - Success      |
| 0x00012019 (73753)    | API (AMQP) | CMD 1008 - Error        |
| 0x0001201a (73754)    | API (AMQP) | CMD 1010 - Success      |
| 0x0001201b (73755)    | API (AMQP) | CMD 1010 - Error        |
| 0x0001201c (73756)    | API (AMQP) | CMD 1012 - Success      |
| 0x0001201d (73757)    | API (AMQP) | CMD 1012 - Error        |
| 0x0001201e (73758)    | API (AMQP) | CMD 1014 - Success      |
| 0x0001201f (73759)    | API (AMQP) | CMD 1014 - Error        |
| 0x00012020 (73760)    | API (AMQP) | CMD 1020 - Success      |
| 0x00012021 (73761)    | API (AMQP) | CMD 1020 - Error        |

##### Avimesa Messages Events - 0x00020000 (131072)

| Event ID (base 16/10) | Origin           | Notes                                        |
| --------------------- | ---------------- | -------------------------------------------- |
| 0x00020001 (131073)   | Avimesa Messages | Device Input parsing error                   |
| 0x00020002 (131074)   | Avimesa Messages | Device Driver Engine error                   |
| 0x00020003 (131075)   | Avimesa Messages | Device Output parsing error                  |
| 0x00020004 (131076)   | Avimesa Messages | Device Firmware Update (DFU) Start Event     |
| 0x00020005 (131077)   | Avimesa Messages | Device Firmware Update (DFU) Progress Update |
| 0x00020006 (131078)   | Avimesa Messages | Device Firmware Update (DFU) Complete Event  |
| 0x00020007 (131079)   | Avimesa Messages | Device Firmware Update (DFU) Error Event     |

[Top][51]  
<a id="5.-dt-quickstart"></a>

## 5\. DialTone QuickStart

<a id="5.1-dt-quickstart"></a>

#### 5\.1 Summary

Please refer to the document Avimesa_DialTone_Protocol.docx for details

<a id="5.2-dt-quickstart"></a>

#### 5\.2 Typical Device Data

The following is provided to give an idea of what typical device data may look like when unaltered and sent from the Device runtime to the queues.

    {  
       "api_maj":0,
       "api_min":11,
       "dts":1536783776,
       "dev":{  
          "dev_id":"000102030405060708090a0b0c0d0e0f",
          "dev_data":{  
             "dev_type":1,
             "fw":1,
             "hw_rev":48,
             "bat":100.00,
             "rssi":0,
             "temp":25.00,
             "dev_sts":1
          },
          "chans":[  
             {  
                "ch_idx":0,
                "ch_data":[  
                   {  
                      "data_idx":0,
                      "units":1,
                      "val":0.003
                   }
                ]
             },
             {  
                "ch_idx":1,
                "ch_data":[  
                   {  
                      "data_idx":0,
                      "units":10,
                      "val":5.0
                   }
                ]
             },
             {  
                "ch_idx":2,
                "ch_data":[  
                   {  
                      "data_idx":0,
                      "units":11,
                      "val":1112014848
                   }
                ]
             }
          ]
       }
    }
    

<a id="5.3-dt-quickstart"></a>

#### 5\.3 Typical Device Configuration

The following is provided to give an idea of what typical device configuration may look like when unaltered.

    {
        "api_maj": 0,
        "api_min": 11,
        "dts": 0,
        "dev": {
            "dev_id": "20010DB800000000026c01fffe5e8584",
            "dev_cfg": {
                "heartbeat": 3
            },
            "chans": [ 
                {
                "ch_idx": 0,
                "ch_cfg": {
                    "ch_type": 1,
                    "en": 1,
                    "sched": 1,
                    "sensor": {
                        "settling_time": 10
                    }
                }
            }, 
            {
                "ch_idx": 8,
                "ch_cfg": {
                    "ch_type": 256,
                    "en": 1,
                    "sched": 1,
                    "sensor": {
                        "sens_flags ": 1
                    }
                }
            }]
        }
    }
    

[Top][51]

 [1]: #1.-summary
 [2]: #1.1-summary
 [3]: #1.2-summary
 [4]: #1.3-references
 [5]: #1.4-terminology
 [6]: #2.-components
 [7]: #2.1-components
 [8]: #2.1.1-components
 [9]: #2.1.2-components
 [10]: #2.1.3-components
 [11]: #2.1.4-components
 [12]: #2.1.5-components
 [13]: #3.-client-usage
 [14]: #3.1-client-usage
 [15]: #3.2-client-usage
 [16]: #3.3-client-usage
 [17]: #3.4-client-usage
 [18]: #3.5-client-usage
 [19]: #4.-group-api
 [20]: #4.1-group-api
 [21]: #4.1.1-group-api
 [22]: #4.1.2-group-api
 [23]: #4.1.3-group-api
 [24]: #4.1.4-group-api
 [25]: #4.1.5-group-api
 [26]: #4.1.6-group-api
 [27]: #4.2-group-api
 [28]: #4.3-group-api
 [29]: #4.4-group-api
 [30]: #4.5-group-api
 [31]: #4.6-group-api
 [32]: #4.7-group-api
 [33]: #4.8-group-api
 [34]: #4.9-group-api
 [35]: #4.10-group-api
 [36]: #4.10.1-group-api
 [37]: #4.10.2-group-api
 [38]: #4.10.3-group-api
 [39]: #4.10.4-group-api
 [40]: #4.11-system-log
 [41]: #4.11.1-system-log
 [42]: #4.11.2-system-log
 [43]: #4.11.3-system-log
 [44]: #5.-dt-quickstart
 [45]: #5.1-dt-quickstart
 [46]: #5.2-dt-quickstart
 [47]: #5.3-dt-quickstart
 [48]: https://www.npmjs.com/package/@avimesa/group-api-amqp
 [49]: https://github.com/Avimesa/sdk-nodejs-group-api-amqp
 [50]: https://github.com/Avimesa/examples-nodejs-group-api-amqp
 [51]: #toc
 [52]: https://raw.githubusercontent.com/Avimesa/user-guide-group-api-amqp/master/images/fig1.png
 [53]: https://raw.githubusercontent.com/Avimesa/user-guide-group-api-amqp/master/images/fig3.png
 [54]: https://raw.githubusercontent.com/Avimesa/user-guide-group-api-amqp/master/images/fig4.png
 [55]: https://raw.githubusercontent.com/Avimesa/user-guide-group-api-amqp/master/images/fig5.png
 [56]: https://raw.githubusercontent.com/Avimesa/user-guide-group-api-amqp/master/images/fig6.png
 [57]: https://github.com/Avimesa/sdk-nodejs-group-api-amqp/blob/develop/lib/rmq.js