# FL5 Attendance & Access Control HTTP Protocol Specification

Attendance, access control, elevator control

Visible Light Terminal Client Communication Protocol Description

Date: 2026/3/13

statement:

Please read the documentation carefully. We assumes no responsibility for data loss caused by improper user operation or other programs.

This manual is designed based on SDK 2.0.

### Description and introduction

The online communication client API is an interface for data communication between internet cloud and online time attendance and access control devices . It facilitates the management of multiple time attendance and access control devices, including user information and fingerprints, downloading attendance records and background operation logs, modifying user information, access control times, device settings, and configuring access control.

The device will save attendance records when it goes offline, and upload the data to the cloud server when it comes back online.

This API includes:

1. Download attendance records.

2. Upload and download user information, card information, fingerprint information, facial information, and palm print information.

3. Configure access control rules for the access control device.

4. View device information, firmware version, number of device attendance records, number of faces and fingerprints that can be registered, recognition methods, etc.

5. View various device events in real time, such as fingerprint verification, personnel check-in, personnel registration, access control status, and facial recognition.

6. Register users directly online and modify user information online.

Configure the time periods and permissions for users to access the access control system.

#### Terminology Explanation

| noun : | explain |
| :--- | :--- |
| machine | Time and attendance/access control devices that communicate with an HTTP server over the Internet |
| Platform Administrator | The server sends instructions to a specific machine ; the platform administrator sends instructions to the machine via a web application on a PC and waits for the execution result. |
| Issue instructions | Instructions issued by the operator to the machine.<br>For example, setting the machine's time or changing the username registered on the machine are examples of operator instructions. |
| Personnel registration data | The data used on the machine to verify and identify the user. Examples include passwords, ID card numbers, fingerprints, facial recognition, and palm prints. |

#### Version History

| time | Version | Update content |
| :--- | :--- | :--- |
| 2023.09.27 |   | 1. Added section 2.9 |
| 2024.11.5 |   | 1. Added the onlineCheck parameter in version 3.2.14<br>Added interface 4.6<br>Revise the usage descriptions in sections 3.2.27 and 4.5. |
| 2025/9/10 |   | 1. Added elevator-controlled door opening method in version 3.2.14<br>2. Added elevator access control rules 3.2.28 |
| 2025/10/17 |   | 3.2.14 Added elevator control mode: elevatorControl<br>Add section 3.2.29<br>Considering the safety risks of elevator operations, the remote elevator start/stop function has been removed. |
| 2026/3/13 |   | Added elevator control MQTT protocol<br>Add sections 5.1.2 and 5.1.3. |

### Communication connection between the machine and the web server

First, 1. The demo was successfully deployed. The machine is connected to the network. During the connection process, the device can be treated as an independent PC. The device must be connected to the network to communicate data.

2. The device's server IP address and port must match the IP address and port of the server you are connecting to. Different connection processes require different device network settings, such as WAN, wireless network, Ethernet connection, etc. Otherwise, the connection may fail. Sometimes, if the device's serial port is busy, you can restart the device.

3. After a successful connection, the web operator can send commands to the machine based on the machine name or machine ID.

#### The communication process between the machine and the web server is divided into two types.

the communication process between the machine and the web server, please refer to Figure 2-1. Communication process during instruction processing .

Communication between machines and servers can be mainly classified into two modes.

1. The process of the server issuing instructions, the machine receiving and executing the instructions issued by the platform administrator.

2. The process of the machine notifying the server that an event has occurred, and the server responding (e.g., changes in user registration data or the creation of new records ) .

#### Using HTTP communication protocol

Explanation: A communication protocol refers to the rules and agreements that two entities must follow to complete communication or services. The protocol defines the format used for data units , the information and meaning that information units should contain, the connection method, and the timing of information transmission and reception, thereby ensuring that data is successfully transmitted to its designated destination within the network .

HTTP communication method:

All requests sent by the machine to the server are  HTTP requests using the POST method. The entire body content is in JSON format , and the binary content within it is entirely composed of base64.

#### The execution process of the web server sending instructions to the machine

The processing procedure is as follows:

The first step is for the platform administrator to select the attendance machine and obtain the machine ID ( device_id).

2. The second step is for the platform administrator to save the instructions that the machine needs to execute in the database linked to the web server .

3. The third step is for the machine to periodically query the server for any instructions sent to it. If there are any, it will execute them and upload the results to the server.

4. The platform administrator queries the execution status of service commands at regular intervals . If a command is marked as executed , the administrator processes the result .

Here, trans_id is the task identifier, which is used to determine which instruction the result corresponds to when returning the execution result of the instruction.

This process is illustrated in Figure 2-1 .

Figure 2-1. Communication flow during instruction processing

If the data packets that the device needs to upload are very large, packet-by-packet transmission may be used. The transmission process is shown in Figure 2-2. For details, please refer to the instructions.

Figure 2-2. Communication flow during instruction processing

Steps 1 and 2 Its records contain the following information.

Task identification number ( trans_id )

Machine identification number ( device_id )

Command identification number ( cmd_code )

Command parameter data ( cmd_param )

Task status ( trans_status )

Last update time of task status ( trans_status_update_time )

#### The format of binary data used in HTTP requests and responses

All binary data is base64 encoded and placed in JSON format.

The time attendance machine implements the requests and responses required to receive instructions from the server.

In order to receive instructions sent to it by the server , the attendance machine sends an HTTP request to the web server at regular intervals and receives a response, with the following specific format:

1. The machine receives requests for instructions from the web server.

In order to receive instructions sent to itself, the time attendance machine sends an HTTP POST request to the web server at regular intervals .

At this point, the HTTP header should contain the following fields .

| field name | Field meaning | Required fields | Limitations | Parameter Description |
| :--- | :--- | :--- | :--- | :--- |
| request _code | Requirement Code | yes | It must be a string like the one below .<br>" receive_cmd " | This indicates that the machine is asking the server if it has sent any instructions to itself. |
| dev_id | Machine Identification Number | yes | English and numbers , maximum 24 characters . | connected to the same web server must have a unique identification number .<br>This parameter refers to the unique number of the time and attendance machine . |
| dev_model | Used to distinguish equipment types | yes | English numbers<br>Maximum 16 bits | Device type; depending on the type, the protocol used by the device may be different. |
| token​ | Device access token | yes | MD5( dev_id) + key) | The manufacturer will issue a unique key to each developer. The corresponding device will then perform an MD5 hash of this key and the device's unique identifier. Developers can verify the accuracy of this signature on the server side to determine the legitimacy of the device. |

For example: HTTP headers are uploaded from the machine to the server:

| POST / HTTP/1.0<br>Accept: image/gif, image/x-xbitmap, image/jpeg, image/pjpeg, application/vnd.ms-excel, application/msword, application/vnd.ms-powerpoint, */*<br>Accept-Language: en-us<br>Accept-Encoding: gzip, deflate<br>User-Agent: Mozilla/4.0<br>Connection: close<br>Content-Length: 201<br>Content-Type: application/octet-stream<br>request_code: receive_cmd<br>dev_id: C2689C470326192F<br>dev_model : V536<br>token: EE5F15B6AF14992F1D9E741397DA6A72 |
| :--- |

The field marked with a BOLD font above is the one that needs attention.

The data placed in the HTTP body section as described in 1.3.

The content of the string placed in the Body section is as follows .

{

" time " : <1> ,​

}

See section 3.2.1 for details on device heartbeat.

When a machine submits an HTTP request to receive instructions, it does not place any binary data in the HTTP body .

#### The server's response to the request to receive instructions.

If the server receives the above request, it checks whether there are any instructions sent to the machine, and if so, sends down a response.

The header and body of the response contain the following information.

The response header contains the following fields .

| field name | Field | Required fields | limit | Parameter Description |
| :--- | :--- | :--- | :--- | :--- |
| response_code | Response Code | yes | English and numeric characters , maximum 64 characters .<br>All English letters are capitalized . | This indicates whether the instruction was received successfully .<br>OK: Success<br>ERROR: Failure<br>ERROR_NO_CMD: No instructions need to be executed. See 3.2.1 for details. |
| trans _id | Task Identifier | no | English and numbers , maximum 16 characters . | The identification number used to track the execution process of instructions . |
| c md_code | Command Identification Number | no | English and numbers , maximum 32 characters.<br>All English letters are capitalized. | A string representing the instruction type.<br>For example: GET_ENROLL_DATA |

Example) The following response is transmitted.

| HTTP/1.1 200 OK<br>Cache-Control: private<br>Server: Microsoft-IIS/7.5<br>Set-Cookie: ASP.NET_SessionId=453lmc45jaelft45glb2mdre; path=/; HttpOnly<br>X-AspNet-Version: 2.0.50727<br>X-Powered-By: ASP.NET<br>Date: Wed, 10 Dec 2014 04:47:42 GMT<br>Connection: close<br>Content-Length: 39<br>Content-Type: application/octet-stream<br>response_code: OK<br>trans_id: 201<br>cmd_code: GET_ENROLL_DATA |
| :--- |

The data placed in the response body varies depending on the instruction.

#### Requests and permissions required for machine to upload command execution results

After the machine executes the instruction, it sends an HTTP request. The request function receives the response and sends its result to the web server, in the following format:

1. Request required for the machine to upload the results of operator command execution.

The machine uploads the following HTTP POST request to the server.

The following fields should be placed in the request header at this time.

| field name | Field meaning | Required fields | limit | Detailed description |
| :--- | :--- | :--- | :--- | :--- |
| request_code | Required code | yes | It must be a string like the one below .<br>" send_cmd_result " | This indicates that the machine uploads the execution result of the command to the server. |
| d ev_id | Machine Identification Number | yes | English and numbers , maximum 24 characters | Reference 2.1.1 |
| dev_model | Distinguish between equipment types | yes | English letters and numbers, maximum 16 digits | Reference 2.1.1 |
| token​ | Device access token | yes | MD5( dev_id + key) | Reference 2.1.1 |
| trans_id | Task Identifier | yes | English and numbers , maximum 16 characters | Reference 2.1.2​ |
| cmd_return_code | Instruction result code | yes | English and numeric characters , maximum 64 bits | This indicates the result of the machine executing the instructions.<br>" OK " indicates successful execution .<br>" ERROR " string or a subsequent string indicating the unexpected occurred during execution. |

The data placed in the body of this request varies depending on the instruction.

#### The server's response to the request for the machine's uploaded results.

The server saves the execution results of the instructions uploaded to the database machine and then sends down the following response.

The response header should contain the following fields.

| field name | Field meaning | Required options | limit | Detailed description |
| :--- | :--- | :--- | :--- | :--- |
| response_code | Response Code | yes | English and numeric characters , maximum 64 characters .<br>The English alphabet is all in large letters . | This indicates that the server has successfully received and saved the result data.<br>OK: Success<br>ERROR: Failure |
| trans _id | Task Identifier | yes | Numbers , maximum 16 digits . | Reference 2.1.2 |

#### Scan to pass (scan the QR code on the mini-program with your device)

Usage process: Users open the QR code corresponding to the mini program, and the access control verifies the personnel information, time information, signature information, and access rules, and grants access. After access is granted, the relevant record data in section 4.1 needs to be uploaded.

QR code format: user ID #current time#signature

userId: The userId issued in section 3.2.4

Current time: yyyyMMddHHmmss, valid for 60 seconds

Signature information: MD5 (current time + person's secret)

secret: The personnel secret issued in section 3.2.4

### Commonly Used Commands

#### Commonly used server-side commands.

| command code | Command Name |
| :--- | :--- |
| GET_DEVICE_INFO | Obtain basic device information |
| GET_USER_ID_LIST | Get User ID List |
| SET_USER_INFO​ | Set user information |
| GET_USER_INFO | Obtaining user information |
| DELETE_ USER | Delete user |
| GET_LOG_DATA | Retrieve machine records by time |
| CLEAR_LOG_DATA​​ | Clear all attendance records on the device |
| SET_TIME | Synchronize device time |
| RESET_FK | Restart the device |
| UPDATE_FIRMWARE | Upgrade device firmware |
| CLEAN MANAGER | One-click removal of device administrator |
| ENTER_ENROLL | Entering registration status |
| SET_DEVICE_SETTING | Set device parameters |
| GET_DEVICE_SETTING | Obtain device parameters |

#### Commonly Used Commands, Machine and Server Request and Response Descriptions

#### Device heartbeat

Obtain the various instructions stored in the machine .

Device request (all subsequent server instructions begin with the device initiating a heartbeat request; the header content is omitted, see section 2.5 for details)

-- HTTP header –

request_code: receive_cmd

dev_id: C2689C470326192F

dev_model : V536

token: EE5F15B6AF14992F1D9E741397DA6A72

-- HTTP body –

{

" time " : " 201910010645 "​

}

Server response (If the server has no instructions to send to the device, it sends an ERROR_NO_CMD response to the device. This response has no body and some header content is omitted; see section 2.6 for details.)

-- HTTP header –

response_code: ERROR_NO_CMD

trans_id:

cmd_code:

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| time | yes | Time when the device sends the request |

#### GET_DEVICE_INFO (Get basic device information)

Purpose: To obtain basic device information for subsequent data processing. The server should automatically retrieve the device's basic information once when the device first connects.

Device heartbeat (see 3.2.1)

Server response (If the server issues a command, the cmd_code header contains the command code. The presence of a body depends on the command content; this command does not have a body).

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code : G ET_ DEVICE_ INFO

Equipment response

-- HTTP header –

request_code: send_cmd_result

trans_id: 100

cmd_return_code : OK

-- HTTP body –

{

" name " : " Device name " ,

" deviceId " : " device ID "

" firmware " : " firmware name " ,

" fp V er " : "FP_128" ,

" f aceV er " : "FACE _D_100 " ,

"pvVer":"PV_100",

"maxBufferLen":92160,

"userLimit":3000,

"fpLimit":10000,

"faceLimit":3000,

"pvLimit":3000,

"pwdLimit":3000,

"cardLimit":3000,

"logLimit":200000,

"userCount":1234,

"managerCount":2,

"fpCount":1111,

"faceCount":123,

"pvCount":123,

"pwdCount":456,

"cardCount":789,

"log quont " : 1234,

" allL og C ount" : 6666,

" supportACL " : 1,

" supportShift " : 0,

}

Server-side confirmation (subsequent server responses to send_cmd_result will follow this pattern, with no body section and the header content omitted; see section 2.8 for details).

-- HTTP header –

response_code : OK

trans_id: 100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| name | yes | Device Name |
| firmware | yes | firmware name |
| fpVer | no | Fingerprint data versions are used for converting fingerprint data between various machine models. |
| faceVer | no | Face data versions are used for converting face data between various machine models. |
| pvVer | no | Palmprint data version , used for converting palmprint data between various machine models. |
| maxBufferLen | yes | Maximum data cache : The maximum size of a data packet that the device can accept. The length of the JSON in the HTTP body sent by the server should be less than this value. |
| user Limit | yes | Maximum number of users |
| f pLimit | no | Maximum number of fingerprints |
| f aceLimit | no | Maximum number of faces |
| pvLimit | no | Maximum number of palm prints |
| pwdLimit | no | Maximum number of passwords |
| cardLimit | no | Maximum number of cards |
| logLimit | yes | Maximum number of records |
| u ser C ount | yes | Current number of users |
| m anager C ount , | yes | Current number of administrators |
| fp C ount | no | Current fingerprint count |
| f aceC ount | no | Current number of faces |
| pvC ount | no | Current number of palm prints |
| p wdC ount | no | Current number of passwords |
| card Coupon​ | no | Current number of cards |
| log C ount | yes | Current record count |
| allL og C ount | yes | Total number of records |
| supportACL | no | Does it support professional access control? |
| supportShift | no | Does it support shift scheduling? |

#### G ET_USER_ID_LIST (Retrieves the list of registered users)

Purpose: To retrieve a list of user IDs registered on the machine.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code:OK

trans_id:100

cmd_code: GET_USER_ID_LIST

Device Response

-- HTTP header --

request_code: send_cmd_result

trans_id:100

cmd_return_code:OK

-- HTTP body --

{

"packageId":1,

"usersCount":1234,

" usersId " :[ " 1 " , " 2 " , " 3 " ,...]

}

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

The process of repeating device response and server confirmation continues until the value of package_id is 0, indicating that data transmission is complete.

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| packageId | yes | The data packet ID is used when the data to be uploaded is too large. The device will then divide the data into packets for transmission. The data packet ID starts from 1 and increments sequentially, with 0 indicating that all data has been transmitted. |
| user sC ount | yes | user IDs |
| u ser sId | no | User ID List |

#### SET_USER_INFO ( Synchronously writes user information to the time and attendance machine )

Purpose: To write data such as fingerprints, faces, palm prints, passwords, card numbers, names, permissions , and user photos of the corresponding ID into the corresponding device.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code:OK

trans_id:100

cmd_code: SET_USER_INFO

-- HTTP body --

{

"users":[{

"userId":"1234",

"name":"John Doe",

"privilege":0,

"card":"123456",

"pwd":"666",

"fps":["base64_0"," base64_1",...," base64_9"],

"face":"base64",

"palm":["base64_0"," base64_1"],

"photo":"base64",

"vaildStart":"yyyyMMdd",

"vaildEnd":"yyyyMMdd",

"update":1,

"photoEnroll":1,

"secret":""

"identityCard":"440183197310203556"

},...]

}

Device Response

-- HTTP header --

request_code: send_cmd_result

trans_id:100

cmd_return_code:OK

-- HTTP body --

{

" usersId " :[ " 1 " , " 2 " , " 3 " ...],

" failCode " : [1,2,3...]

}

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| users | yes | When sending out user arrays , be sure to determine the number of users sent each time based on the device's maximum buffer size. |
| user ID | yes | User ID |
| name | no | username |
| privilege | no | User permissions : 0 represents a regular user, 1 represents an administrator. |
| card​ | no | card number |
| pwd | no | password |
| fps | no | Fingerprint data array , binary data , represented using base64 . |
| face | no | Face data , binary data , represented using base64 . |
| palm | no | Palmprint data array , binary data , represented using base64 . |
| photo | no | User photos, binary data , represented using base64 . |
| vaildStart | no | User validity period start date |
| vaildEnd | no | User validity period end date |
| update | no | 0: The original ID's data will be deleted and replaced with the new data.<br>1: The original ID's relevant data will be saved, only overwriting the corresponding data sent out.<br>If this item does not exist, the default value is 0. |
| photoEnroll | no | 1: Use photo to register face; 0: Do not use. The default value is 0 if this option does not exist.<br>If this value is 1, the sent face data will be ignored, the original face data in the device will be deleted, and the sent photo will be used to register the face. |
| s ecret | no | The visitor QR code key is a 32-bit hexadecimal string . The purpose and usage of the visitor QR code are as follows:<br>Example of plaintext information in a QR code :<br>{ "userId":"123"," begin ":" 202205220910 ","end":" 202205231120 "," visitId ": 9876, " check " :0 or 1 ,"sign":"md5" }<br>Signature rules:<br>MD5( userId \| begin \| end \| visitId \| check \| secret )<br>Example: md5 (123 \|202205220910\|202205231120\|9876\|1\| secret )<br>3. The employee forwards the QR code to the visitor. The visitor holds the QR code and goes to the access control system. The access control system identifies the QR code according to the algorithm in steps 1 and 2. If the identification fails, the error reason is displayed. If the identification is successful, the check field is checked. If check is 0, the door is opened directly, and the corresponding interface in section 4.4 is called to upload the visitor access record. If check is 1, the interface in section 4.5 is called. After receiving an OK return, the corresponding interface in section 4.4 is called to upload the visitor access record. After receiving an ERROR return, the visitor communication failure is indicated. |
| usersId | no | User IDs that failed to be sent |
| failCode | no | For details of the reasons for the failure to send the data, please refer to the attached table. |
| identityCard | no | Issuing ID card numbers |

#### GET_USER_INFO ( Retrieves user registration information from the time and attendance machine)

Purpose: To retrieve user registration information from the corresponding attendance machine, such as fingerprints, facial recognition, palm prints, passwords, ID cards, names, photos, and permissions. Because the amount of data to be retrieved may be large, the data needs to be sent in packets (see Figure 2-2 in section 2.3 for the flowchart). Note that if any of the sent user IDs do not exist in the device, the number of users returned will be less than the size of the requested ID array.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: GET_USER_INFO​​

-- HTTP body -- ( usersId can be empty , but the body field cannot be empty , i.e., it can be an empty JSON object {})

{

" usersId " :[ " 1 " , " 2 " , " 3 " ...]

}

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id: 100

cmd_return_code:OK

-- HTTP body --

{

"packageId":1,

"usersCount":5

"users":[ {

"userId":"1",

"name":"John Doe",

"privilege":0

"card":"123456"

"pwd":"666"

"fps":["base64_0"," base64_1",...," base64_9"],

"face":"base64",

"palm":["base64_0"," base64_1"],

"photo":"base64",

"vaildStart":"yyyyMMdd",

"vaildEnd":"yyyyMMdd",

"timeGroups":{

"sun":["HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm"],

"mon":["HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm"],

"tue":["HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm"],

"wed":["HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm"],

"thu":["HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm"],

"fri":["HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm"],

" sat " :HmmHHmm " , " HHmmHHmm " , " HHmmHHmm " , " HHmmHHmm " , " HHmmHHmm " , " HHmmHHmm " ]

}

},...]

}

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

The process of repeating device response and server confirmation continues until the value of package_id is 0, indicating that data transmission is complete.

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| usersId | no | The desired array of user IDs; if this field does not exist , it is considered to retrieve all user data . |
| packageId | yes | The data packet ID is used when the data to be uploaded is too large. The device will then divide the data into packets for transmission. The data packet ID starts from 1 and increments sequentially, with 0 indicating that all data has been transmitted. |
| usersCount | yes | The number of users included in this package's data. |
| users | yes | The device responds to the user array , with the same user structure as users in 3.2.4. |

#### DELETE_USER (Delete Person )

Purpose: To delete the registration information for the corresponding ID from the device. For some device models, deleting a user's information will also delete the user's associated attendance records.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: DELETE_ USER

-- HTTP body --

{

" usersCount " : 100,

"usersId":["1","2","3"...]

}

Device Response

-- HTTP header --

request_code: send_cmd_result

trans_id:100

cmd_return_code:OK

-- HTTP body --

{

" usersId " :[ " 1 " , " 2 " , " 3 " ...]

}

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| usersCount | yes | The number of users to be deleted should be the same as the size of the users_id array.<br>If the quantity is 0, it means you want to delete all data, and the users_id array can be omitted. |
| usersId | no | The server sends an array of user IDs to be deleted. The client responds with an array of user IDs indicating that the deletion failed. |

#### GET_LOG_DATA (Retrieves log data)

Retrieve machine-recorded data for a specific time period from the machine.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: GET_LOG_DATA​​

-- HTTP body – ( beginTime and endTime can be empty , but the body cannot be empty , meaning it can be an empty JSON object {} )

{

"newLog":0 , " begin T ime ":" yyyyMMdd "," end T ime ":" yyyyMMdd ","clearMark":0

}

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id:100

cmd_return_code:OK

-- HTTP body --

{

"packageId":0,

"allLogCount":26,

"logsCount":26

"logs":[{

"userId":"1",

"time":"yyyyMMddHHmmss",

"verifyMode":"Card+Face",

"ioMode":1,

"inOut":"In",

"doorMode":"hand_open",

"temperature":37.1,

"logPhoto":"Base64",

"IDCard":{

"ID":"123456202001011234",

"name":"John Doe",

"birthDay":"yyyyMMdd",

"address":"AAABBB",

"nation":1,

"sex":1,

"photo":"Base64",

}

},...]

}

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

The process of repeating device response and server confirmation continues until the value of package_id is 0, indicating that data transmission is complete.

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| newLog | no | Should a new record be created? 1: Yes, the time period will be ignored and record flags will be cleared; 0: No, the default is 0. |
| begin Time​ | no | Start time ; if this field does not exist, it represents all data up to the end time. |
| endTime | no | End time ; if this field does not exist, it represents all data after the start time. |
| clearMark | no | Whether to clear the record flag for this time period. 0: No, 1: Yes, default is 0. |
| packageId | yes | The data packet ID is used when the data to be uploaded is too large. The device will then divide the data into packets for transmission. The data packet ID starts from 1 and increments sequentially, with 0 indicating that all data has been transmitted. |
| allLogsCount | yes | The total number of records that need to be uploaded. |
| logsCount | yes | Number of records contained in this package |
| L ogs | yes | The array of users who responded to the device . |
| userId | yes | Check-in User ID |
| T ime | yes | Check-in time |
| verifyMode | yes | Check-in methods include fingerprint and facial recognition . |
| ioMode | no | Check-in mode |
| inOut | no | Entry/Exit Mode |
| doorMode | no | Opening mode |
| temperature | no | Check-in temperature |
| logPhoto | no | Check-in photos , binary data , base64 format |
| IDCard | no | ID card information JSON object |
| ID | no | ID number |
| name | no | Name on ID Card |
| birthDay | no | Birthday on ID card |
| address | no | ID card address |
| nation | no | ethnicity on ID card |
| sex | no | ID card gender |
| photo | no | ID photo |

#### CLEAR_LOG_DATA (Delete all record data)

Delete all attendance records on the device.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: CLEAR_LOG_DATA

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id: 100

cmd_return_code : OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

This command has no parameters.

#### SET_TIME (synchronization time)

Synchronize device time.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: SET_TIME

-- HTTP body --

{

" syncTime " : " yyyyMMddHHmmss "

}

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id: 100

cmd_return_code : OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| syncTime | yes | The time the server wants to synchronize |

#### RESET_FK (Machine restart)

Send this command when the attendance machine needs to be restarted for some reason .

restarting , the machine will ignore all previously executed instructions and enter a state of receiving new instructions .

This command does not include a device response or server confirmation process and does not carry any parameters.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: RESET_FK

#### UPDATE_FIRMWARE (Update Firmware)

Notify the device to update the firmware. Note that the device should not be powered off during the upgrade process.

Device heartbeat (see 3.2.1)

Server response  (Note that the key value " firmwareFIleURL " uses uppercase "i" and lowercase "L" due to historical errors)

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: UPDATE_FIRMWARE

-- HTTP body --

{

" firmwareFIleURL " : " www.xxx.com/xxxx.bin "

}

Equipment response

The device does not respond further to this instruction . When the server receives the device's request for the URL, it indicates that the device has received the server's instruction and is in the process of downloading and updating firmware.

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| firmwareFIleURL | yes | The firmware URL is the address to which the device will receive the update request. Upon receiving this URL, the device will stop sending heartbeats and download the new firmware to the target address. After the update is complete, it will automatically reboot. Note that the device's response only indicates that the URL has been received, not that the upgrade was successful. The server can use the `GET_DEVICE_INFO` command to query the firmware version and determine whether the device upgrade was successful. |

#### CLEAR_MANAGER (Clear device administrator)

Clearing the administrator badge on the device will not delete the administrator's user information.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: CLEAR_MANAGER

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id: 100

cmd_return_code : OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

This command has no parameters.

#### ENTER _ ENROLL ( Remotely access the registration interface )

Access the registration interface by issuing a command.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: ENTER_ENROLL

-- HTTP body --

{

userId : " 666 " ,​​

" face "​​​​

}

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id: 100

cmd_return_code : OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| userId | yes | User ID to register |
| f eature | yes | The feature types to register are : face (human face), fp (fingerprint), card (card), pwd (password), and palm (palm print). |

#### SET_DEVICE_SETTING (Set device parameters )

Send out various device setting parameters

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: SET_DEVICE_SETTING

-- HTTP body – (See parameter list for detailed settings)

{

" xxx " : " xxx " ,

" xxxx " :{

" xx " : " xx "

}

}

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id: 100

cmd_return_code : OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter explanation (All fields in this command are optional, and the final value is a string)

| Field |   | illustrate |
| :--- | :--- | :--- |
| dev Name |   | Device Name |
| w iegand Type |   | Wiegand (26/34) |
| serverHost |   | Server address. Note that modifying this setting will change the polling address, and the original server will no longer receive subsequent commands. |
| serverPort |   | Server port: Note that modifying this setting will change the polling port, and the original server will no longer receive subsequent commands. |
| pushServerHost |   | Push server address |
| pushServerPort |   | Push server port |
| pushEnable |   | Enable real-time push notifications (yes/no) |
| i nterval |   | Heart rate interval (seconds) |
| l anguage |   | Device language (selectable depending on firmware type, such as CHS/CHT/en/es/es-AR, see appendix for details) |
| v olume |   | Device volume (1~10) |
| a nti Pass |   | Enable anti-submarine warfare (Yes/No) |
| open DoorDelay |   | Door opening delay (seconds) |
| t amper Alarm |   | Enable tamper alarm (yes/no) |
| alarmDelay |   | Alarm delay (seconds) |
| reverifyTime |   | Repeat confirmation time (minutes) |
| screensaversTime |   | How long after the main interface remains unchanged will it enter screensaver mode (in seconds)? 0 indicates the screensaver is off. |
| s leepTime |   | How long after the main interface remains unchanged will it enter sleep mode (in minutes)? 0 indicates sleep mode is off. |
| v erifyMode |   | Check-in method selection (see appendix for a list of options) |
| living |   | Enable/disable liveness detection (yes/no/double) or (lv0: disable liveness detection ; lv1: intermediate liveness detection ; lv2: advanced liveness detection ) |
| inOut |   | Access control machine status (in/out) |
| MultiCheck |   | Simultaneous confirmation number (1~10) |
| matchDistance |   | Recognition distance (near/mid/far) |
| matchThreshold |   | Recognition threshold (10~80) |
| live Threshold |   | Recognition threshold (10~80) |
| a dmin Pwd | oldPwd | Sets the web server password; `oldPwd` represents the previous password, and `newPwd` is the new password. |
|   | newPwd |   |
| bdgAddress |   | Address , number of buildings, number of rooms. For example: Building 11, 12 rooms (maximum 24 bytes). |
| companyDpm |   | Company information , such as : XX Company XX Department (maximum 48 bytes) |
| onlineCheck |   | The online access verification switch is a boolean value. When true, if the device's local access rules do not support door opening, it calls interface 4.6 online access verification. If it returns OK , the door is opened; otherwise, it returns ERROR, the door is not opened. |
| elevatorType |   | Ladder control mode, string type:<br>allPass: Allows access to all users (no restrictions on the number of people or floors).<br>userPass: Whitelisted users can open doors (each person on the device can open doors on each floor).<br>userRulePass: Rules for opening doors for whitelisted personnel (each person is issued floor and time instructions individually, see section 3.2.28; each person can have multiple time periods and floors).<br>Special Note: Administrators can open any floor at any time. |
| elevatorControl |   | Elevator control mode, boolean type:<br>1: Control<br>0: No control; controlled by the elevator itself. |

#### GET_DEVICE_SETTING ( Get device parameters )

Obtain various settings parameters from the device as needed.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: GET_DEVICE_SETTING

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id: 100

cmd_return_code : OK

-- HTTP body – (The returned parameter list is the same as SET_DEVICE_SETTING)

{

" xxx " : " xxx " ,

" xxxx " :{

" xx " : " xx "

}

}

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

Same as SET_DEVICE_SETTING

#### RESET_DEVICE (Restore factory settings )

Restore most of the factory settings

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: RESET_DEVICE

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id: 100

cmd_return_code : OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

This command has no parameters.

#### SET_DOOR_STATUS (Sets the status of the gate relay )

Set the state of the door control relay to control the opening and closing of the door.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: SET_DOOR_STATUS

-- HTTP body --

{

" doorStatus " : " open_close "

}

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id: 100

cmd_return_code : OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| doorStatus | yes | close: Close; open: Open ( does not close automatically ) ; open_close: Open, automatically closes after a delay. |

#### RESET_ENROLL_MARK (Resets the real-time registration data flag )

Clear the real-time registration data flag by issuing an instruction.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: RESET_ENROLL_MARK

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id: 100

cmd_return_code : OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

#### RESET_LOG_MARK (Resets real-time attendance data flags )

Clear the real-time attendance data flags for a certain time period by issuing an instruction.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: RESET_LOG_MARK

-- HTTP body -- ( beginTime and endTime can be empty , but the body cannot be empty , meaning it can be an empty JSON object { } )

{

" begin T ime ":" yyyyMMdd "," end T ime ":" yyyyMMdd "

}

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id: 100

cmd_return_code : OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| begin Time​ | no | Start time ; if this field does not exist, it represents all data up to the end time. |
| endTime | no | End time ; if this field does not exist, it represents all data after the start time. |

#### SET_USERS_TIME_GROUPS (Sets the user access control time period )

Set the user's access control time period by issuing instructions.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: SET_USERS_TIME_GROUPS

-- HTTP body --

{

"usersId":["1","2","3"...],

"timeGroups":{

"sun":["HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm"],

"mon":["HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm"],

"tue":["HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm"],

"wed":["HHmmHHmm","HHmmHHmm","HHmmHHmm"],

"thu":["HHmmHHmm","HHmmHHmm"],

"fri":["HHmmHHmm"]

}

}

Device Response

-- HTTP header --

request_code: send_cmd_result

trans_id:100

cmd_return_code:OK

-- HTTP body --

{

" usersId " :[ " 1 " , " 2 " , " 3 " ...]

}

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| usersId | no | The desired array of user IDs. If this field does not exist , it will be used to set access control data for all users within the device . |
| t imeGroups | yes | to be set |
| sun | no | The Sunday time slot can contain a maximum of 6 strings representing the time slot (start time to end time). The array length can be less than 6 strings. Missing strings will default to " 00000000 " . If this field does not exist, all strings will be " 00000000 " , meaning it is not allowed on this day. It is available all day and can be filled with [ " 00002359 " ] . |
| m on | no | Monday session, other times as above |
| tue | no | Tuesday session, other times as above |
| w ed | no | Wednesday session, other times as above |
| thu | no | Thursday session, other times as above |
| fri | no | Friday session, other procedures are the same as above. |
| sat | no | Saturday session, otherwise the same as above. |

#### SET_AUTHKEY ( Issuing authorization information )

Devices that require card reading sectors can use this interface to send authorization information.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: SET_AUTHKEY

-- HTTP body --

{

" key " : " 79853C9C3CBF326EF77A8CD92675BD82766AD13FE932B10575514E267633A097 " ,

" cardSector " : 1

}

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id: 100

cmd_return_code : OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| key | yes | Authorization information (24 or 96-bit hexadecimal string, 96 for reading via authorization card) |
| cardSector | yes | Card sector number (0-15) |

#### OPEN DOOR ( Remote door opening/ closing )

the door by issuing commands

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: OPEN_DOOR

-- HTTP body --

{

" opType ": " open " , // open (on), close (off), keep_open (always open), keep_close (always closed), cancel_keep (cancel always open/always closed)

" op Time " : " yyyyMMddHHmmss "// Message sending time

" expireSeconds ": 60 // Valid seconds, 60 (default value, numeric type)

}

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id: 100

cmd_return_code : OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| opType | yes | open, close, keep_open, keep_close, cancel_keep |
| op Time | yes | The message was sent at time " yyyyMMddhhmmss " . |
| expireSeconds | yes | Valid seconds, 10 (default value, numeric type) |

#### SET_BELL_DATA ( Send ring )

Get device ring settings data

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code : SET_BELL_DATA

-- HTTP body --

{

" bellList":[

{"no":"1","time":"0000","valid":"0"},

{"no":"2","time":"0000","valid":"0"},

{"no":"24","time":"0000","valid":"0"}

]

}

Device Response

-- HTTP header --

request_code: send_cmd_result

trans_id:100

cmd_return_code:OK

Server Acknowledgment (see Section 2.8)

-- HTTP header --

response_code:OK

trans_id:100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| bellList | yes | Alarm list |
| no | yes | Alarm number |
| time | yes | Ringing time, HHmm |
| valid | yes | Ring count: 0 indicates not enabled. |

#### GET_BELL_DATA (Get ringtone )

Devices that require card reading sectors can use this interface to send authorization information.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id:100

cmd_code: GET_BELL_DATA

{

"bellList":[

{"no":"1","time":"0000","valid":"0"},

{"no":"2","time":"0000","valid":"0"},

{"no":"24","time":"0000","valid":"0"}

]

}

Device Response

-- HTTP header --

request_code: send_cmd_result

trans_id:100

cmd_return_code : OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| bellList | yes | Alarm list |
| No | yes | Alarm number |
| Time | yes | Ringing time, HHmm |
| Valid | yes | Ring count: 0 indicates not enabled. |

#### SET_MQTT_PARAM (Send MQTT connection information)

Configure device MQTT connection information

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code:OK

trans_id:100

cmd_code:SET_MQTT_PARAM

-- HTTP body --

{

IP address : 192.0.2.10​​

" port ": 1883,

" userName ":" mqttUser " ,

" pwd ":" 111111 " ,

" serverTopic ":" kq /server "

}

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id:100

cmd_return_code :OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id:100

Parameter Explanation

| Field | Required | Data types | illustrate |
| :--- | :--- | :--- | :--- |
| IP | yes | string | MQTT server IP or domain name |
| port | yes | int | MQTT service port |
| userName | yes | string | mqtt login username |
| pwd | yes | string | mqtt login password |
| serverTopic | yes | string | The cloud service subscription message topic defaults to kq /server. |

#### SET_PHOTO_FILE ( Send ad image )

use: Send advertising images to the device.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code:OK

trans_id:100

cmd_code: SET_PHOTO_FILE

-- HTTP body --

{

"data":[

{

"photo":"PhotoBase64",

},

{

"photo":"PhotoBase64",

},

]

}

Device Response

-- HTTP header --

request_code: send_cmd_result

trans_id:100

cmd_return_code :OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id:100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| photo | yes | PhotoBase64 , advertising images, JPG format, supports up to 10 images . |

#### SET_KEEP_PARAMS ( Sets the parameters for automatic long-term on/off operation )

use: Set automatic long-on/long-off parameters

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: SET_KEEP_PARAMS​​​

-- HTTP body --

{

type: " keep_open or keep_close, keep_off " // Normally open/normally closed type keep_off disables normally open

rule:{"sun":["HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm"],

"mon":["HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm"],

"tue":["HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm"],

"wed":["HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm"],

"thu":["HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm"],

"fri":["HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm"],

"sat":HmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm","HHmmHHmm"]}

}

Device Response

-- HTTP header --

request_code: send_cmd_result

trans_id:100

cmd_return_code:OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id:100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| keep_open | no | The long-open parameter is not set; otherwise, no long-open information is displayed. |
| keep_close | no | The "stay on" parameter, if not set, is considered as having no "stay off" information. |
| sun | no | Sunday time slot: This is a string array of unlimited length, defaulting to an empty string. It can be filled in all day [ " 00002359 " ]. If not set, it means there is no long open/long close information for this time slot. |
| m on | no | Monday session, other times as above |
| tue | no | Tuesday session, other times as above |
| w ed | no | Wednesday session, other times as above |
| thu | no | Thursday session, other times as above |
| fri | no | Friday session, other procedures are the same as above. |
| sat | no | Saturday session, otherwise the same as above. |

#### SET_USER_ELEVATOR_RULE (Set elevator access rules )

Elevator access rules are set by issuing instructions.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: SET_USER_ELEVATOR_RULE

-- HTTP body -- ( beginTime and endTime can be empty , but the body cannot be empty , meaning it can be an empty JSON object { } )

{

" userIds ": [ " 123456 "," 456 " , … ],

"ruleType":"clear", // clear, add, remove

"setRules":[{

ruleId: 123

beginDate: "yyyyMMdd",

endDate: "yyyyMMdd",

passTimes:["HHmmHHmm","HHmmHHmm"]

layers: [1,2,3,-1]

},{

ruleId: 124

beginDate: " yyyyMMdd " ,

endDate: " yyyyMMdd " ,

passTimes:[ " HHmmHHmm " , " HHmmHHmm " ]

layers: [1,2,3,-1]

}],

removeRuleIds:[1,2,3]}

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id: 100

cmd_return_code : OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| userIds | no | The personnel registration number array; if the array is empty , it's to handle the default rule that all employees have the same number. |
| ruleType | yes | clear: Removes all communication rules for the user.<br>add: Add a common rule to setRules<br>remove: Removes all rules from removeRuleIds. |
| setRules | no | When a new access rule is added, the added access rule item is appended to each person's information. |
| removeRuleIds | no | Remove the specified RuleId, the parameter is a long array. |

#### SET_ELEVATOR_ LAYERS (Sets the elevator floor display )

floor display by issuing a command.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: SET_ELEVATOR_ LAYERS

-- HTTP body -- ( However , the body field cannot be empty ; it can be an empty array [])

[{

"floor": 1, "display":"B1", "control": 1

},

"floor": 2, "display":"F1", "control": 0

},{

" floor " : 3, " display " : " A2 " , " control " : 1

}]

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id: 100

cmd_return_code : OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| floor | yes | The physical floors correspond to floors 1 to 64. Floors not filled in are considered not under control. |
| display | yes | The floor display can be filled in according to the floor number. For example, the 3rd floor can be displayed as F3; the 4th floor can be displayed as A3. |
| control |   | Elevator floor control mode, boolean type:<br>1: Control<br>0: No control; controlled by the elevator itself. |

#### Transmission (Transmission 485 )

By sending a command to pass through 485, the machine is instructed to send commands to 485 according to the server's instructions.

Device heartbeat (see 3.2.1)

server response

-- HTTP header –

response_code: OK

trans_id: 100

cmd_code: TRANSMISSION

-- HTTP body -- ( However , the body field cannot be empty ; it can be an empty array [])

{

"transmission":"ABCD" }

}

Equipment response

-- HTTP header --

request_code: send_cmd_result

trans_id: 100

cmd_return_code : OK

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| transmission | yes | Corresponding 485 instruction content |

### Real-time data transmission function

notification of the WEB server when certain events occur on the device . Real-time transmission is the attendance machine actively sending requests to the WEB server and receiving responses .

This includes the ability to record and transmit data in real time .

### 4.1 Real-time attendance record transmission

Real-time recording and transmission refers to the function of actively pushing new records to the server when they are generated on the machine .

Device push

-- HTTP header --

request_code: realtime_g log​

-- HTTP body --

{

" u ser I d":"1",

"time":"yyyyMMddHHmmss",

"verifyMode":"Card+Face",

"ioMode":1,

"inOut":"In",

"doorMode":"hand_open",

"temperature":37.1,

"logPhoto":"Base64"

}

Server Acknowledgment (see Section 2.8)

-- HTTP header --

response_code:OK

trans_id: 100

Parameter Explanation

This command pushes one check-in record at a time, with the same parameters as in 3.2.7.

### 4.2 Real-time door status

Real-time door status refers to the status of the door sensor detected by the device being transmitted to the server in real time.

Device push

-- HTTP header --

request_code: realtime_door_status​

-- HTTP body --

{

" doorStatus ":" open "

}

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| doorStatus | yes | Door sensor status : open: open, close: closed |

### 4.3 Real-time registration and data transmission

Real-time registration data transmission is a function where the machine sends registration data to the server whenever the registration data is updated on the device . Data sent by the server will not trigger this push.

Device push

-- HTTP header --

request_code: realtime_enroll_data

-- HTTP body --

{

"userId":"1",

"name":"John Doe",

"privilege":0

"card":"123456"

"pwd":"666"

"fps":["base64_0"," base64_1",...," base64_9"],

"face":"base64",

"palm":["base64_0"," base64_1"]

"photo":"base64"

"vaildStart":"yyyyMMddHHmmss"

" vaildEnd " : " yyyyMMddHHmmss "

}

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

Parameter Explanation

This command pushes one piece of user data at a time, with parameters the same as in 3.2.5.

### 4.4 Visitor Access Data

Device push

-- HTTP header --

request_code: door_visit

-- HTTP body --

{

" user ID " : " 1 " ,​

" visitId " : 1 , // Access identifier, the same as the access identifier (int type) in Section 3.2.4 .

" time " : " yyyyMMddHHmmss "

}

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

### 4.5 Visitor Online Verification

Device push

-- HTTP header --

request_code: door_visit_check

-- HTTP body --

{

" visitId " : 1 , // Access identifier, the same as the access identifier (int type) in Section 3.2.4 .

}

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK OR ERROR

### 4.6 Online Access Verification

Device push

-- HTTP header --

request_code: pass_online_check

-- HTTP body --

{

" userId " : " 123 " , // Registration number

}

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK OR ERROR

### 4.7 Operation Log Data Transmission

Operation log data transmission is the process of sending operation logs to the server when a user interacts with the machine.

Device push

-- HTTP header --

request_code: Opera_glog

-- HTTP body --

{

" u ser I d":"1",

" time ":"yyyyMMddHHmmss",

"operationId":"1",

"operationValue":"",

}

Server-side confirmation (see 2.8 for details)

-- HTTP header --

response_code : OK

trans_id: 100

| Field | Required | illustrate |
| :--- | :--- | :--- |
| u ser I d | yes | ID of the operation |
| operationId | yes | Operation ID , see attached table for details. |
| time | yes | Operation time |
| operationValue | no | Reserved |

### .mqtt protocol

The MQTT protocol is currently mainly used by servers to send remote door opening and closing commands to devices. Devices connect to the MQTT service, and the parameters for obtaining subscription information are referenced in SET_MQTT_PARAM. The subscription topic for each device is: kq /{deviceId}. After obtaining and executing the command, the device calls interface 5.2 to upload the processing result.

mqtt command format:

{

" transId " : Instruction sequence number, data type: long.

" cmd_code " : Command type, Data type: string

" cmd_param " : Command parameters: Data parameters in JSON format

}

#### The server issues an MQTT command.

#### OPEN_DOOR (Remote door opening/closing)

cmd_code: open_door

cmd_param:

{

" opType ": " open " , // open (on), close (off), keep_open (always open), keep_close (always closed), cancel_keep (cancel always open/always closed)

" op Time " : " yyyyMMddHHmmss "// Message sending time

" expireSeconds ": 15 // Valid seconds, 15 (default value, numeric type)

}

#### SET_USER_ELEVATOR_RULE (Sets elevator access rules, equivalent to 3.2.28 )

cmd_code: SET_USER_ELEVATOR_RULE

cmd_param:

{

"userIds":["123456","456",…] ,

"ruleType":"clear", // clear, add, remove

"setRules":[{

ruleId: 123

beginDate: "yyyyMMdd",

endDate: " yyyyMMdd " ,

passTimes:[ " HHmmHHmm " , " HHmmHHmm " ]

layers: [1,2,3,-1]

},{

ruleId: 124

startDate: " yyyyMMdd " ,

endDate: " yyyyMMdd " ,

passTimes:[ " HHmmHHmm " , " HHmmHHmm " ]

layers: [1,2,3,-1]

}],

removeRuleIds:[1,2,3]}

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| userIds | no | The personnel registration number array; if the array is empty , it's to handle the default rule that all employees have the same number. |
| ruleType | yes | clear: Removes all communication rules for the user.<br>add: Add a common rule to setRules<br>remove: Removes all rules from removeRuleIds. |
| setRules | no | When a new access rule is added, the added access rule item is appended to each person's information. |
| removeRuleIds | no | Remove the specified RuleId, the parameter is a long array. |

#### SET_ELEVATOR_ LAYERS (Sets the elevator floor display, equivalent to 3.2.29 )

cmd_code: SET_ELEVATOR_LAYERS

cmd_param:

[{

"floor": 1, "display":"B1", "control": 1

},

"floor": 2, "display":"F1", "control": 0

},{

" floor " : 3, " display " : " A2 " , " control " : 1

}]

Parameter Explanation

| Field | Required | illustrate |
| :--- | :--- | :--- |
| floor | yes | The physical floors correspond to floors 1 to 64. Floors not filled in are considered not under control. |
| display | yes | The floor display can be filled in according to the floor number. For example, the 3rd floor can be displayed as F3; the 4th floor can be displayed as A3. |
| control |   | Elevator floor control mode, boolean type:<br>1: Control<br>0: No control; controlled by the elevator itself. |

#### Device uploads MQTT command processing results

After the device completes the MQTT processing, it sends the processing result to the server topic (serverTopic as defined in section 3.2.25).

mqtt command format:

{

" transId " : 123456,

" deviceId " : " serial number "

" token " : " xxxxxxx " ,

" cmd_return_code " : " OK "

}

| field name | Field meaning | Required fields | Data types | Detailed description |
| :--- | :--- | :--- | :--- | :--- |
| transId | Instruction number | yes | long​ | transId when issuing commands |
| deviceId | Device Identification Number | yes | string | English and numbers , maximum 24 characters |
| token | Device access token | yes | string | MD5(dev_id + key) |
| cmd_return_code | Instruction result code | yes | string | English and numeric characters , maximum 64 bits<br>This indicates the result of the device executing the instructions.<br>" OK " indicates successful execution .<br>" ERROR " string or a subsequent string indicating the unexpected occurred during execution. |

### Version update log

| Update date | Update Notes | Remark |
| :--- | :--- | :--- |
| 2018-11-6 | First draft |   |
| 2020-01-13 | Increased the total number of machine information records; added DoorMode records; added device initialization commands; added name to the all personnel ID retrieval command, with some parameter changes; added real-time monitoring of switch settings to the parameter setting command; added remote door control commands. |   |
| 2020-04-30 | Two new commands have been added to reset the real-time upload flag. |   |
| 2020-05-26 | We revised some problematic wording descriptions and added some device setting parameters. |   |
| 2020-07-04 | An access control time period interface has been added, divided into user-based and batch-based distribution sections; some text errors in this document have been corrected. |   |
| 2020-08-03 | The interface for the palm print section has been added and modified. |   |
| 2020-09-11 | Some additional text descriptions have been added. |   |
| 2020-10-16 | The description of the online upgrade command that received no further responses has been corrected, and a bug has been flagged. |   |
| 2021-01-19 | Some deviceSetting settings have been added, and some instructions have been modified. |   |
| 2022-03-02 | Add the SET_AUTHKEY command |   |
| 2022.05.22 | The new visitor offline and log upload agreement includes the following sections:<br>Add section 3.2.25<br>Added section 4.4 |   |
| 2022-07-01 | Add 3.2.23 and 3.2.24. |   |
| 2022-07-23 | Change the doorkey to secret in SET_USER_INFO |   |
| 2022-07-26 | Modify SET_USER_INFO to add an online/offline detection field.<br>Added visitor online verification function in section 4.5 |   |
| 2022-10-19 | Added 3.2.25 SET_MQTT_PARAM to send MQTT connection information.<br>Add 5.mqtt protocol |   |

Appendix :

| operation_id | Operation Name |
| :--- | :--- |
| 1 | Add personnel |
| 2 | Editors |
| 3 | Delete personnel |
| 4 |   |
| 5 |   |
| 6 | Delete all records |
| 7 | Delete all users |
| 8 | Remove all administrator privileges |
| 9 | Delete all data |
| 10 |   |
| 11 |   |
| 12 |   |
| 13 |   |
| 14 | Check-in records |
| 15 | Register to view |
| 16 | Equipment Information |
| 17 |   |
| 18 |   |
| 19 |   |
| 20 | Download on USB drive |
| twenty one | Download via USB drive --> Report download |
| twenty two | USB flash drive download --> User download |
| twenty three | Download via USB drive --> Schedule download |
| twenty four | Upload to USB drive |
| 25 | USB drive upload --> User upload |
| 26 | USB drive upload --> Scheduled upload |
| 27 | Firmware upgrade |
| 28 |   |
| 29 |   |
| 30 |   |
| 31 |   |
| 32 |   |
| 33 | Simultaneous confirmation number |
| 34 | Door opening delay |
| 35 | Door sensor type |
| 36 | Gate magnetic delay |
| 37 | Anti-escape |
| 38 | Alarm delay |
| 39 | Cancel alarm |
| 40 | Anti-tamper alarm |
| 41 | Fire alarm |
| 42 | Wiegand settings |
| 43 | Wiegand settings --> Wiegand format |
| 44 | Wiegand Settings --> Wiegand Output |
| 45 | Wiegand settings --> Wiegand input |
| 46 | Access control machine status |
| 47 | Serial port settings |
| 48 | Mask testing |
| 49 | Saving illegal logs |
| 50 | QR code recognition mode |
| 51 | Remote password |
| 52 |   |
| 53 |   |
| 54 |   |
| 55 |   |
| 56 |   |
| 57 |   |
| 58 | Departmental Structure |
| 59 | Scheduling |
| 60 | Attendance Rules |
| 61 | Attendance rules --> Allowed late time |
| 62 | Attendance rules --> Allowed early departure time |
| 63 | Identification settings |
| 64 | Identification Settings --> Confirmation Method |
| 65 | Identification Settings --> Repeat Confirmation Time |
| 66 | Recognition Settings --> Continuous Recognition |
| 67 | Recognition Settings --> Continuous Recognition |
| 68 | Identification Settings --> Strangers Record |
| 69 | Recognition Settings --> Attendance Status |
| 70 | Recognition Settings --> Workcode |
| 71 | Identification Settings --> Download Original Record Encryption |
| 72 |   |
| 73 |   |
| 74 |   |
| 75 |   |
| 76 |   |
| 77 |   |
| 78 | Machine Name |
| 79 | Ethernet |
| 80 | Ethernet --> Allow |
| 81 | Ethernet --> Auto Acquisition |
| 82 | Ethernet --> MAC |
| 83 | Ethernet --> IP address |
| 84 | Ethernet --> Subnet Mask |
| 85 | Ethernet --> Default Gateway |
| 86 | Ethernet --> DNS Server |
| 87 | Wireless settings |
| 88 | local area network |
| 89 | Local Area Network --> Port Number |
| 90 | Local Area Network (LAN) --> Communication Password |
| 91 | Wide Area Network |
| 92 | WAN --> Server IP |
| 93 | WAN --> Server Port |
| 94 | WAN --> Request Interval |
| 95 | Push settings |
| 96 | Push Settings --> Push Server IP |
| 97 | Push Settings --> Push Server Port |
| 98 | Push settings --> Send push notifications |
| 99 | Display IP address |
| 100 | https |
| 101 |   |
| 102 |   |
| 103 |   |
| 104 |   |
| 105 |   |
| 106 |   |
| 107 |   |
| 108 |   |
| 109 |   |
| 110 |   |
| 111 | Date and Time |
| 112 | Date and Time --> Date |
| 113 | Date and Time --> Time |
| 114 | Date and Time --> Time Zone |
| 115 | Date and Time --> Automatic Time Synchronization |
| 116 | Date and Time --> Daylight Saving Time |
| 117 | Date and Time --> Daylight Saving Time Start Time |
| 118 | Date and time --> Daylight Saving Time End Time |
| 119 | Display settings |
| 120 | Display Settings --> Interface Return Time |
| 121 | Display settings --> Screen saver |
| 122 | Display settings --> Sleep time |
| 123 | Display Settings --> LCD Brightness |
| 124 | Fill light |
| 125 | Fill light --> Fill light settings |
| 126 | Fill light --> Fill light brightness |
| 127 | Fill light --> Fill light sensitivity |
| 128 | Sound settings |
| 129 | Sound settings --> Notification sounds |
| 130 | Sound settings --> Volume settings |
| 131 | Ring settings |
| 132 | Biometric parameters |
| 133 | Biometric parameters --> Liveness detection |
| 134 | Biometric parameters --> 1:N matching threshold |
| 135 | Biometric parameters --> Liveness detection threshold |
| 136 | Biometric parameters --> Recognition distance |
| 137 | Language settings |
| 138 | Restart the machine |
| 139 | Factory reset |
| 140 |   |
| 141 |   |
| 142 |   |
| 143 |   |
| 144 |   |
| 145 |   |

#### 2. result_code

| value | illustrate |
| :--- | :--- |
| -5 | Equipment capacity full |
| -4 | Communication error |
| -3 | Timeout error |
| -2 | Parameter error |
| -1 | mistake |
| 0 | success |
| 1 | Equipment busy |
| 2 | Equipment in operation |
| 3 | Equipment completed |

#### 3. FailCode

| value | illustrate |
| :--- | :--- |
| 1 | userId format incorrect |
| 2 | The image format was incorrect. |
| 3 | Photo registration error |
| 4 | Storage error |
| 5 | Error in sending facial data |
| 6 | Face data storage error |
| 7 | Fingerprint data delivery error |
| 8 | Error in sending palmprint data |
| 9 | Duplicate registration (registration information already exists on another user's account) |

#### 4. verifyMode

| value | illustrate |
| :--- | :--- |
| 1 | Individual Attendance Mode |
| 2 | Face or Finger or Card or Password or Plam |
| 3 | Password + (Face or Finger) |
| 4 | Card + (Face or Finger) |
| 5 | (Password or Card) + (Face or Finger) |
| 6 | Finger + Face |
| 7 | Face + Finger |
| 8 | Card + Finger + Password |
| 9 | Card + Finger + Face |
| 10 | Password |
| 11 | Card |
| 12 | Finger |
| 13 | Face |
| 14 | Plam |

#### 5 doorMode

| value | illustrate |
| :--- | :--- |
| close_door | close the door |
| hand_open | Manual door opening |
| remote_open | Remote door opening |
| remote_close | Remote door closing |
| Illegal_open | Unauthorized opening of the door |
| Illegal_close | illegal closing |
| Schedule_open | Forced door opening |
| Schedule_close | Forced door closed |
| open | Normal door opening |
| fire_open | Opening the door during a fire |
| door_open_timeout | Door open for too long (door left open for an extended period) |
| door_magnet_open | The door is open (indicating that the door is open, detected by the door magnetic sensor). |
| door_magnet_close | Door magnetic closure (indicating the door is closed, detected by the door magnetic sensor) |
