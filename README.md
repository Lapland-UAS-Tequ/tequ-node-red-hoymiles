# tequ-node-red-hoymiles
Read data from Hoymiles HMS microinverter in Node-RED using Hoymiles Modbus protocol.

Hoymiles HMS-1000-T microinverter data is read directly from DTU-PRO-S RS485 port. You need to setup Hoymiles DTU unit to "Remote Control/Modbus Protocol"-mode and set known Modbus address to DTU with Hoymiles installer application. RS485 adapter used is EasySync ES-U-3001-M and flow is developed and tested with Windows 10 machine and Raspberry PI 3 B+.

Default baudrate is 9600 bps. In this example Modbus address 254 is used.

Starting register for microinverter Modbus data is 0x1000 = 4096. Each inverter PV port data is 40 registers long. HMS-1000-T has two ports. Port 1 data can be read starting from register 4096 and port 2 data can be read starting from register 4096 + 40 = 4136. If there there more microinverters their data will be starting 4096 + (40 * n) register.

## Install required nodes

Modbus communication is implemented using 

https://flows.nodered.org/node/node-red-contrib-modbus

Data is parsed using:

https://flows.nodered.org/node/node-red-contrib-buffer-parser

```
npm install node-red-contrib-modbus
npm install node-red-contrib-buffer-parser
```

## Example flow

```
[{"id":"2eec3c4e45304a6b","type":"modbus-flex-getter","z":"860a95c57f7d2947","name":"","showStatusActivities":true,"showErrors":false,"logIOActivities":false,"server":"f2783f1226f5760b","useIOFile":false,"ioFile":"","useIOForPayload":false,"emptyMsgOnFail":false,"keepMsgProperties":true,"x":650,"y":180,"wires":[["2a1644284073ba29"],[]]},{"id":"be5776b2c5759892","type":"function","z":"860a95c57f7d2947","name":"Create Modbus requests","func":"// Single HMS-1000-T \n// Two ports / inverter\nlet port_count = 2;\nlet modbus_requests = []\n\nfor(let i=0;i<port_count;i++){\n    newRequest = {\n        \"payload\":{\n            'fc': 3, \n            'unitid': 254, \n            'address': 4096+(i*40), \n            'quantity': 40\n        }\n    }\n    modbus_requests.push(newRequest)\n    \n}\n\nreturn [modbus_requests];","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":390,"y":180,"wires":[["2eec3c4e45304a6b"]]},{"id":"a8e98c348b4662a7","type":"inject","z":"860a95c57f7d2947","name":"read data","props":[{"p":"payload"},{"p":"topic","vt":"str"}],"repeat":"","crontab":"","once":false,"onceDelay":0.1,"topic":"","payload":"","payloadType":"date","x":180,"y":180,"wires":[["be5776b2c5759892"]]},{"id":"2a1644284073ba29","type":"buffer-parser","z":"860a95c57f7d2947","name":"Decode Hoymiles microinverter data packet","data":"payload","dataType":"msg","specification":"spec","specificationType":"ui","items":[{"type":"uint8","name":"data_type","offset":0,"length":1,"offsetbit":0,"scale":"1","mask":""},{"type":"hex","name":"serial_number","offset":1,"length":6,"offsetbit":0,"scale":"1","mask":""},{"type":"uint8","name":"port_number","offset":7,"length":1,"offsetbit":0,"scale":"1","mask":""},{"type":"uint16be","name":"pv_voltage","offset":8,"length":1,"offsetbit":0,"scale":"0.1","mask":""},{"type":"uint16be","name":"pv_current","offset":10,"length":1,"offsetbit":0,"scale":"0.01","mask":""},{"type":"uint16be","name":"grid_voltage","offset":12,"length":1,"offsetbit":0,"scale":"0.1","mask":""},{"type":"uint16be","name":"grid_frequency","offset":14,"length":1,"offsetbit":0,"scale":"0.01","mask":""},{"type":"uint16be","name":"pv_power","offset":16,"length":1,"offsetbit":0,"scale":"0.1","mask":""},{"type":"uint16be","name":"today_production","offset":18,"length":1,"offsetbit":0,"scale":"0.1","mask":""},{"type":"uint32be","name":"total_production","offset":20,"length":1,"offsetbit":0,"scale":"0.1","mask":""},{"type":"int16be","name":"temperature","offset":24,"length":1,"offsetbit":0,"scale":"0.1","mask":""},{"type":"uint16be","name":"operating_status","offset":26,"length":1,"offsetbit":0,"scale":"1","mask":""},{"type":"uint16be","name":"alarm_code","offset":28,"length":1,"offsetbit":0,"scale":"1","mask":""},{"type":"uint16be","name":"alarm_count","offset":30,"length":1,"offsetbit":0,"scale":"1","mask":""},{"type":"uint8","name":"link_status","offset":32,"length":1,"offsetbit":0,"scale":"1","mask":""},{"type":"uint8","name":"fixed_value","offset":33,"length":1,"offsetbit":0,"scale":"1","mask":""},{"type":"uint8","name":"reserved_1","offset":34,"length":1,"offsetbit":0,"scale":"1","mask":""},{"type":"uint8","name":"reserved_2","offset":35,"length":1,"offsetbit":0,"scale":"1","mask":""},{"type":"uint8","name":"reserved_3","offset":36,"length":1,"offsetbit":0,"scale":"1","mask":""},{"type":"uint8","name":"reserved_4","offset":37,"length":1,"offsetbit":0,"scale":"1","mask":""},{"type":"uint8","name":"reserved_5","offset":38,"length":1,"offsetbit":0,"scale":"1","mask":""},{"type":"uint8","name":"reserved_6","offset":40,"length":1,"offsetbit":0,"scale":"1","mask":""}],"swap1":"","swap2":"","swap3":"","swap1Type":"swap","swap2Type":"swap","swap3Type":"swap","msgProperty":"payload","msgPropertyType":"str","resultType":"keyvalue","resultTypeType":"return","multipleResult":false,"fanOutMultipleResult":false,"setTopic":true,"outputs":1,"x":290,"y":260,"wires":[["f2a11a875dcce5d4"]]},{"id":"f2a11a875dcce5d4","type":"debug","z":"860a95c57f7d2947","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"payload","targetType":"msg","statusVal":"","statusType":"auto","x":630,"y":260,"wires":[]},{"id":"f2783f1226f5760b","type":"modbus-client","name":"Hoymiles_modbus","clienttype":"simpleser","bufferCommands":true,"stateLogEnabled":false,"queueLogEnabled":false,"tcpHost":"127.0.0.1","tcpPort":"502","tcpType":"DEFAULT","serialPort":"/dev/ttyUSB0","serialType":"RTU-BUFFERD","serialBaudrate":"9600","serialDatabits":"8","serialStopbits":"1","serialParity":"none","serialConnectionDelay":"100","unit_id":"254","commandDelay":"1000","clientTimeout":"1000","reconnectOnTimeout":true,"reconnectTimeout":"2000","parallelUnitIdsAllowed":true}]
```

## Sources:
Hoymiles - Technical note - Modbus implementation using 3Gen DTU-Pro

