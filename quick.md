- **Step 1: Start all of the containers**

```bash
docker-compose up -d
```

- **Step 2: Initialize the replica sets (config servers and shards) and routers**

```bash
docker-compose exec configsvr01 sh -c "mongo < /scripts/init-configserver.js"
docker-compose exec shard01-a sh -c "mongo < /scripts/init-shard01.js"
docker-compose exec shard02-a sh -c "mongo < /scripts/init-shard02.js"
docker-compose exec shard03-a sh -c "mongo < /scripts/init-shard03.js"
```

- **Step 3: Initializing the router**

```bash
docker-compose exec router01 sh -c "mongo < /scripts/init-router.js"
```

- **Step 4: Enable sharding and setup sharding-key**

```bash
docker-compose exec router01 mongo --port 27017

// Enable sharding for database `genieacs`
sh.enableSharding("genieacs")
// Setup shardingKey for collection `devices`**
db.adminCommand( { shardCollection: "genieacs.devices", key: { _id: "hashed" } } )
```

- **Verify the status of the sharded cluster**

```bash
docker-compose exec router01 mongo --port 27017

sh.status()
```

- **Verify status of replica set for each shard**

```bash
docker exec -it rydell-shard-01-node-a bash -c "echo 'rs.status()' | mongo --port 27017" 
docker exec -it rydell-shard-02-node-a bash -c "echo 'rs.status()' | mongo --port 27017" 
docker exec -it rydell-shard-03-node-a bash -c "echo 'rs.status()' | mongo --port 27017" 
```

- **Check the distribution of collection**

```bash
docker-compose exec router01 mongo --port 27017

db.adminCommand( { flushRouterConfig: "genieacs.devices" } );
db.getSiblingDB("genieacs").devices.getShardDistribution();

db.getSiblingDB("genieacs").faults.getShardDistribution();

```


# make zone configuration

```bash

docker-compose exec router01 mongo --port 27017

db.getSiblingDB("genieacs2").devices.createIndex({"_deviceId._Manufacturer": 1, _id:"hashed"},{unique:false})
```

```bash
docker-compose exec router01 mongo --port 27017


sh.addShardToZone("rs-shard-01", "KV1")
sh.addShardToZone("rs-shard-02", "KV2")
sh.addShardToZone("rs-shard-03", "KV3")
```

```bash
docker-compose exec router01 mongo --port 27017

sh.updateZoneKeyRange(
  "genieacs2.devices",
  { "_deviceId._Manufacturer" : "Dasan Network Solutions,. INC", "_id" : MinKey },
  { "_deviceId._Manufacturer" : "Dasan Network Solutions,. INC", "_id" : MaxKey },
  "KV3"
)

sh.updateZoneKeyRange(
	"genieacs2.devices",
	{ "_deviceId._Manufacturer" : "Huawei,. INC","_id" : MinKey },
	{ "_deviceId._Manufacturer" : "Huawei,. INC","_id" : MaxKey },
	"KV1"
)


sh.updateZoneKeyRange(
	"genieacs2.devices",
	{ "_deviceId._Manufacturer" : "ZTE,. INC","_id" : MinKey },
	{ "_deviceId._Manufacturer" : "ZTE,. INC","_id" : MaxKey },
	"KV2"
)

```

```bash
docker-compose exec router01 mongo --port 27017

sh.enableSharding("genieacs2")
sh.shardCollection("genieacs2.devices",{ "_deviceId._Manufacturer":1, _id:"hashed"}, false)
```

```bash
docker-compose exec router01 mongo --port 27017

use genieacs2

use genieacs2;

for (let i = 0; i < 100; i++) {
	db.devices.insert(
	{
		_id: 'd096fb-H646GM-00000' + i,
		InternetGatewayDevice: {
			DeviceInfo: {
				HardwareVersion: {
					_object: false,
					_timestamp: ISODate('2021-08-20T16:21:46.158Z'),
					_type: 'xsd:string',
					_value: 'DS-E5-583-A1'
				},
				ProvisioningCode: {
					_object: false,
					_timestamp: ISODate('2021-08-20T16:21:46.158Z'),
					_type: 'xsd:string',
					_value: ''
				},
				SoftwareVersion: {
					_object: false,
					_timestamp: ISODate('2021-08-20T16:21:46.158Z'),
					_type: 'xsd:string',
					_value: 'VT5.2.3070118'
				},
				SpecVersion: {
					_object: false,
					_timestamp: ISODate('2021-08-20T16:21:46.158Z'),
					_type: 'xsd:string',
					_value: '1.0'
				},
				_object: true
			},
			ManagementServer: {
				ConnectionRequestURL: {
					_object: false,
					_timestamp: ISODate('2021-08-20T16:21:46.158Z'),
					_type: 'xsd:string',
					_value: 'http://192.168.1.151:64524/'
				},
				ParameterKey: {
					_object: false,
					_timestamp: ISODate('2021-08-20T16:21:46.158Z'),
					_type: 'xsd:string',
					_value: ''
				},
				_object: true
			},
			WANDevice: {
				'1': {
					WANConnectionDevice: {
						'1': {
							WANIPConnection: {
								'1': {
									ExternalIPAddress: {
										_object: false,
										_timestamp: ISODate('2021-08-20T16:21:46.158Z'),
										_type: 'xsd:string',
										_value: '172.3.89.139'
									},
									_object: true
								},
								_object: true
							},
							WANPPPConnection: {
								'1': {
									ExternalIPAddress: {
										_object: false,
										_timestamp: ISODate('2021-08-20T16:21:46.158Z'),
										_type: 'xsd:string',
										_value: '192.168.4.204'
									},
									_object: true
								},
								_object: true
							},
							_object: true
						},
						_object: true
					},
					_object: true
				},
				_object: true
			},
			_object: true
		},
		_deviceId: {
			_Manufacturer: 'Dasan Network Solutions,. INC',
			_OUI: 'd096fb',
			_ProductClass: 'H646GM',
			_SerialNumber: '00000' + i
		},
		_lastInform: ISODate('2021-08-20T16:21:46.157Z'),
		_registered: ISODate('2021-08-20T16:21:46.157Z')
	}
	)
}
```