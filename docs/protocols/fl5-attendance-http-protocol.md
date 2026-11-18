# FL5 Attendance & Access Control HTTP Protocol Specification

Attendance, access control, elevator control
Visible Light Terminal Client Communication Protocol Description
Date: 2026/3/13
statement:
Please read the documentation carefully. We assumes no responsibility for data loss caused by improper user operation or other programs.
This manual is designed based on SDK 2.0.
Table of contents
Table of Contents 3

## 1. Description and Introduction


## 1.1 Terminology Explanation 2

Version 1.2 History 2

## 2. Communication connection between the machine and the web server 3.


## 2.1 The communication process between the machine and the Web server is divided into two types .


## 2.2 Using the HTTP communication protocol 4


## 2.3 Execution process of instructions sent from the web server to the machine 4


## 2.4 Format of binary data used in HTTP requests and responses 8


## 2.5 The attendance machine implements the requests and responses required to receive server instructions .


## 2.6 Server's response to a request for a receiving instruction (10)


## 2.7 Requests and Approvals Required for Uploading Machine Instruction Execution Results 12


## 2.8 Server's response to the machine's upload result request 13


## 2.9 Scan to Access (Scan the access QR code on the mini-program using your device) 13


## 3. 15 Commonly Used Commands


## 3.1 Commonly used server-side commands. 15


## 3.2 Commonly Used Commands: Machine and Server Request and Response Explanation 16


## 3.2.1 Equipment heartbeat 16


## 3.2.2 GET_DEVICE_INFO (Get basic device information) 17


## 3.2.3 G ET_USER_ID_LIST (Retrieves the list of registered users) 20


## 3.2.4 SET_USER_INFO ( Synchronously writes user information to the time and attendance machine ) 22


## 3.2.5 GET_USER_INFO ( Retrieves user registration information from the time and attendance machine) 26


## 3.2.6 DELETE_USER (Deleted Person ) 28


## 3.2.7 GET_LOG_DATA (Retrieves record data) 31


## 3.2.8 CLEAR_LOG_DATA (Delete all record data) 35


## 3.2.9 SET_TIME (Synchronization Time) 37


## 3.2.10 RESET_FK (Machine restart) 38


## 3.2.11 UPDATE_FIRMWARE (Firmware Update) 39


## 3.2.12 CLEAR_MANAGER (Clear Device Administrator) 40


## 3.2.13 ENTER _ ENROLL ( Remotely access the registration interface ) 41


## 3.2.14 SET_DEVICE_SETTING (Set device parameters ) 42


## 3.2.15 GET_DEVICE_SETTING ( Get Device Parameters ) 46


## 3.2.16 RESET_DEVICE (Factory Reset ) 48


## 3.2.17 SET_DOOR_STATUS (Sets the status of the gate relay ) 49


## 3.2.18 RESET_ENROLL_MARK (Reset real-time registration data flag ) 50


## 3.2.19 RESET_LOG_MARK (Reset real-time check-in data flag ) 51


## 3.2.20 SET_USERS_TIME_GROUPS (Sets the user access control time period ) 52


## 3.2.21 SET_AUTHKEY ( Issuing authorization information ) 55


## 3.2.22 OPEN _ DOOR ( Remote door opening/closing ) 56


## 3.2.23 SET_BELL_DATA ( Ring signal ) 57


## 3.2.24 GET_BELL_DATA (Get Ringtone ) 59


## 3.2.25 SET_MQTT_PARAM (Send MQTT connection information) 61


## 3.2.26 SET_PHOTO_FILE ( Send ad image ) 62


## 3.2.27 SET_KEEP_PARAMS ( Sets automatic long - on / long-off parameters ) 64


## 3.2.28 SET_USER_ELEVATOR_RULE (Set elevator access rules ) 66


## 3.2.29 SET_ELEVATOR_ LAYERS (Set elevator floor display ) 68


## 3.2.30 Transmission (Transparent Transmission 485 ) 70


## 4. Real-time data transmission function 72


## 4.1 Real-time attendance record transmission 73


## 4.2 Real-time door status 74


## 4.3 Real-time registration and data transmission 75


## 4.4 Visitor Access Data 76


## 4.5 Visitor Online Verification 77


## 4.6 Online access verification 77


## 4.7 Operation Log Data Transmission 78


## 5. MQTT protocol 80


## 5.1 The server issues the MQTT command 80


## 5.1.1 OPEN_DOOR (Remote Door Opening/Closing) 81


## 5.1.2 SET_USER_ELEVATOR_RULE (Sets elevator access rules, equivalent to 3.2.28 ) 81


## 5.1.3 SET_ELEVATOR_ LAYERS (Sets the elevator floor display, equivalent to 3.2.28 ) 83


## 5.2 Device upload MQTT command processing result 84


## 6. Version Update Log 85


## 2. result_code 93


## 3. FailCode 94


## 4. verifyMode 94

5-door Mode 95
Description and introduction
The online communication client API is an interface for data communication between internet cloud and online time attendance and access control devices . It facilitates the management of multiple time attendance and access control devices, including user information and fingerprints, downloading attendance records and background operation logs, modifying user information, access control times, device settings, and configuring access control.
The device will save attendance records when it goes offline, and upload the data to the cloud server when it comes back online.
This API includes:

## 1. Download attendance records.


## 2. Upload and download user information, card information, fingerprint information, facial information, and palm print information.


## 3. Configure access control rules for the access control device.


## 4. View device information, firmware version, number of device attendance records, number of faces and fingerprints that can be registered, recognition methods, etc.


## 5. View various device events in real time, such as fingerprint verification, personnel check-in, personnel registration, access control status, and facial recognition.


## 6. Register users directly online and modify user information online.

Configure the time periods and permissions for users to access the access control system.
Terminology Explanation

| noun : | explain |
| :--- | :--- |
| machine | Time and attendance/access control devices that communicate with an HTTP server over the Internet |
| Platform Administrator | The server sends instructions to a specific machine ; the platform administrator sends instructions to the machine via a web application on a PC and waits for the execution result. |
| Issue instructions | Instructions issued by the operator to the machine. For example, setting the machine's time or changing the username registered on the machine are examples of operator instructions. |
| Personnel registration data | The data used on the machine to verify and identify the user. Examples include passwords, ID card numbers, fingerprints, facial recognition, and palm prints. |

Version History

| time | Version | Update content |
| :--- | :--- | :--- |
| 2023.09.27 | - | 1. Added section 2.9 |
| 2024.11.5 | - | 1. Added the onlineCheck parameter in version 3.2.14 Added interface 4.6 Revise the usage descriptions in sections 3.2.27 and 4.5. |
| 2025/9/10 | - | 1. Added elevator-controlled door opening method in version 3.2.14 2. Added elevator access control rules 3.2.28 |
| 2025/10/17 | - | 3.2.14 Added elevator control mode: elevatorControl Add section 3.2.29 Considering the safety risks of elevator operations, the remote elevator start/stop function has been removed. |
| 2026/3/13 | - | Added elevator control MQTT protocol Add sections 5.1.2 and 5.1.3. |

Communication connection between the machine and the web server
First, 1. The demo was successfully deployed. The machine is connected to the network. During the connection process, the device can be treated as an independent PC. The device must be connected to the network to communicate data.

## 2. The device's server IP address and port must match the IP address and port of the server you are connecting to. Different connection processes require different device network settings, such as WAN, wireless network, Ethernet connection, etc. Otherwise, the connection may fail. Sometimes, if the device's serial port is busy, you can restart the device.


## 3. After a successful connection, the web operator can send commands to the machine based on the machine name or machine ID.

The communication process between the machine and the web server is divided into two types.
the communication process between the machine and the web server, please refer to Figure 2-1. Communication process during instruction processing .
Communication between machines and servers can be mainly classified into two modes.

## 1. The process of the server issuing instructions, the machine receiving and executing the instructions issued by the platform administrator.


## 2. The process of the machine notifying the server that an event has occurred, and the server responding (e.g., changes in user registration data or the creation of new records ) .

Using HTTP communication protocol
Explanation: A communication protocol refers to the rules and agreements that two entities must follow to complete communication or services. The protocol defines the format used for data units , the information and meaning that information units should contain, the connection method, and the timing of information transmission and reception, thereby ensuring that data is successfully transmitted to its designated destination within the network .
HTTP communication method:
All requests sent by the machine to the server are HTTP requests using the POST method. The entire body content is in JSON format , and the binary content within it is entirely composed of base64.
The execution process of the web server sending instructions to the machine
The processing procedure is as follows:
The first step is for the platform administrator to select the attendance machine and obtain the machine ID ( device_id).

## 2. The second step is for the platform administrator to save the instructions that the machine needs to execute in the database linked to the web server .


## 3. The third step is for the machine to periodically query the server for any instructions sent to it. If there are any, it will execute them and upload the results to the server.


## 4. The platform administrator queries the execution status of service commands at regular intervals . If a command is marked as executed , the administrator processes the result .

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
The format of binary data used in HTTP requests and responses
All binary data is base64 encoded and placed in JSON format.
The time attendance machine implements the requests and responses required to receive instructions from the server.
In order to receive instructions sent to it by the server , the attendance machine sends an HTTP request to the web server at regular intervals and receives a response, with the following specific format:

## 1. The machine receives requests for instructions from the web server.

In order to receive instructions sent to itself, the time attendance machine sends an HTTP POST request to the web server at regular intervals .
At this point, the HTTP header should contain the following fields .

| field name | Field meaning | Required fields | Limitations | Parameter Description |
| :--- | :--- | :--- | :--- | :--- |
| request _code | Requirement Code | yes | It must be a string like the one below . " receive_cmd " | This indicates that the machine is asking the server if it has sent any instructions to itself. |
| dev_id | Machine Identification Number | yes | English and numbers , maximum 24 characters . | connected to the same web server must have a unique identification number . This parameter refers to the unique number of the time and attendance machine . |
| dev_model | Used to distinguish equipment types | yes | English numbers Maximum 16 bits | Device type; depending on the type, the protocol used by the device may be different. |
| token​ | Device access token | yes | MD5( dev_id) + key) | The manufacturer will issue a unique key to each developer. The corresponding device will then perform an MD5 hash of this key and the device's unique identifier. Developers can verify the accuracy of this signature on the server side to determine the legitimacy of the device. |

For example: HTTP headers are uploaded from the machine to the server:

| POST / HTTP/1.0 Accept: image/gif, image/x-xbitmap, image/jpeg, image/pjpeg, application/vnd.ms-excel, application/msword, application/vnd.ms-powerpoint, */* Accept-Language: en-us Accept-Encoding: gzip, deflate User-Agent: Mozilla/4.0 Connection: close Content-Length: 201 Content-Type: application/octet-stream request_code: receive_cmd dev_id: C2689C470326192F dev_model : V536 token: EE5F15B6AF14992F1D9E741397DA6A72 |
| :--- |

The field marked with a BOLD font above is the one that needs attention.
The data placed in the HTTP body section as described in 1.3.
The content of the string placed in the Body section is as follows .
{
" time " : <1> ,​
}
See section 3.2.1 for details on device heartbeat.
When a machine submits an HTTP request to receive instructions, it does not place any binary data in the HTTP body .
The server's response to the request to receive instructions.
If the server receives the above request, it checks whether there are any instructions sent to the machine, and if so, sends down a response.
The header and body of the response contain the following information.
The response header contains the following fields .

| field name | Field | Required fields | limit | Parameter Description |
| :--- | :--- | :--- | :--- | :--- |
| response_code | Response Code | yes | English and numeric characters , maximum 64 characters . All English letters are capitalized . | This indicates whether the instruction was received successfully . OK: Success ERROR: Failure ERROR_NO_CMD: No instructions need to be executed. See 3.2.1 for details. |
| trans _id | Task Identifier | no | English and numbers , maximum 16 characters . | The identification number used to track the execution process of instructions . |
| c md_code | Command Identification Number | no | English and numbers , maximum 32 characters. All English letters are capitalized. | A string representing the instruction type. For example: GET_ENROLL_DATA |

Example) The following response is transmitted.

| HTTP/1.1 200 OK Cache-Control: private Server: Microsoft-IIS/7.5 Set-Cookie: ASP.NET_SessionId=453lmc45jaelft45glb2mdre; path=/; HttpOnly X-AspNet-Version: 2.0.50727 X-Powered-By: ASP.NET Date: Wed, 10 Dec 2014 04:47:42 GMT Connection: close Content-Length: 39 Content-Type: application/octet-stream response_code: OK trans_id: 201 cmd_code: GET_ENROLL_DATA |
| :--- |

The data placed in the response body varies depending on the instruction.
Requests and permissions required for machine to upload command execution results
After the machine executes the instruction, it sends an HTTP request. The request function receives the response and sends its result to the web server, in the following format:

## 1. Request required for the machine to upload the results of operator command execution.

The machine uploads the following HTTP POST request to the server.
The following fields should be placed in the request header at this time.

| field name | Field meaning | Required fields | limit | Detailed description |
| :--- | :--- | :--- | :--- | :--- |
| request_code | Required code | yes | It must be a string like the one below . " send_cmd_result " | This indicates that the machine uploads the execution result of the command to the server. |
| d ev_id | Machine Identification Number | yes | English and numbers , maximum 24 characters | Reference 2.1.1 |
| dev_model | Distinguish between equipment types | yes | English letters and numbers, maximum 16 digits | Reference 2.1.1 |
| token​ | Device access token | yes | MD5( dev_id + key) | Reference 2.1.1 |
| trans_id | Task Identifier | yes | English and numbers , maximum 16 characters | Reference 2.1.2​ |
| cmd_return_code | Instruction result code | yes | English and numeric characters , maximum 64 bits | This indicates the result of the machine executing the instructions. " OK " indicates successful execution . " ERROR " string or a subsequent string indicating the unexpected occurred during execution. |

The data placed in the body of this request varies depending on the instruction.
The server's response to the request for the machine's uploaded results.
The server saves the execution results of the instructions uploaded to the database machine and then sends down the following response.
The response header should contain the following fields.

| field name | Field meaning | Required options | limit | Detailed description |
| :--- | :--- | :--- | :--- | :--- |
| response_code | Response Code | yes | English and numeric characters , maximum 64 characters . The English alphabet is all in large letters . | This indicates that the server has successfully received and saved the result data. OK: Success ERROR: Failure |
| trans _id | Task Identifier | yes | Numbers , maximum 16 digits . | Reference 2.1.2 |

Scan to pass (scan the QR code on the mini-program with your device)
Usage process: Users open the QR code corresponding to the mini program, and the access control verifies the personnel information, time information, signature information, and access rules, and grants access. After access is granted, the relevant record data in section 4.1 needs to be uploaded.
QR code format: user ID #current time#signature
userId: The userId issued in section 3.2.4
Current time: yyyyMMddHHmmss, valid for 60 seconds
Signature information: MD5 (current time + person's secret)
secret: The personnel secret issued in section 3.2.4
Commonly Used Commands
Commonly used server-side commands.

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

Commonly Used Commands, Machine and Server Request and Response Descriptions
Device heartbeat
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

GET_DEVICE_INFO (Get basic device information)
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

G ET_USER_ID_LIST (Retrieves the list of registered users)
Purpose: To retrieve a list of user IDs registered on the machine.
Device heartbeat (see 3.2.1)
server response
-- HTTP header –
response_code:OK
trans_id:100
cmd_code: GET_USER_ID_LIST
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

SET_USER_INFO ( Synchronously writes user information to the time and attendance machine )
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
"name":" ",
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
| update | no | 0: The original ID's data will be deleted and replaced with the new data. 1: The original ID's relevant data will be saved, only overwriting the corresponding data sent out. If this item does not exist, the default value is 0. |
| photoEnroll | no | 1: Use photo to register face; 0: Do not use. The default value is 0 if this option does not exist. If this value is 1, the sent face data will be ignored, the original face data in the device will be deleted, and the sent photo will be used to register the face. |
| s ecret | no | The visitor QR code key is a 32-bit hexadecimal string . The purpose and usage of the visitor QR code are as follows: Example of plaintext information in a QR code : { "userId":"123"," begin ":" 202205220910 ","end":" 202205231120 "," visitId ": 9876, " check " :0 or 1 ,"sign":"md5" } Signature rules: MD5( userId \| begin \| end \| visitId \| check \| secret ) Example: md5 (123 \|202205220910\|202205231120\|9876\|1\| secret ) 3. The employee forwards the QR code to the visitor. The visitor holds the QR code and goes to the access control system. The access control system identifies the QR code according to the algorithm in steps 1 and 2. If the identification fails, the error reason is displayed. If the identification is successful, the check field is checked. If check is 0, the door is opened directly, and the corresponding interface in section 4.4 is called to upload the visitor access record. If check is 1, the interface in section 4.5 is called. After receiving an OK return, the corresponding interface in section 4.4 is called to upload the visitor access record. After receiving an ERROR return, the visitor communication failure is indicated. |
| usersId | no | User IDs that failed to be sent |
| failCode | no | For details of the reasons for the failure to send the data, please refer to the attached table. |
| identityCard | no | Issuing ID card numbers |

GET_USER_INFO ( Retrieves user registration information from the time and attendance machine)
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
"name":" ",
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

DELETE_USER (Delete Person )
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
| usersCount | yes | The number of users to be deleted should be the same as the size of the users_id array. If the quantity is 0, it means you want to delete all data, and the users_id array can be omitted. |
| usersId | no | The server sends an array of user IDs to be deleted. The client responds with an array of user IDs indicating that the deletion failed. |

GET_LOG_DATA (Retrieves log data)
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
"name":" ",
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

CLEAR_LOG_DATA (Delete all record data)
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
SET_TIME (synchronization time)
Synchronize device time.
Device heartbeat (see 3.2.1)

> *End of extracted summary — see full revision history on SUNTEK Developer Portal.*

