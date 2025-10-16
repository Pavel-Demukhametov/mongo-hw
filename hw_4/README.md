# ДЗ 4

ФИО: Демухаметов Павел

## Окружение
- Windows 11 + Docker
- Образ: mongo:8.0.14

## Создаём сеть и поднимаем docker compose: конфиг rsConfig (cfg1–cfg3, порты 27001–27003); два шарда  — rsShard1 (sh1n1–sh1n3, 27011–27013) и rsShard2 (sh2n1–sh2n3, 27021–27023); два mongos (27000 и 27010)

~~~sh
docker network create mongo-net
docker compose up -d
~~~


## Создаём репликосеты для конфигов
~~~sh
docker exec -it cfg1 mongosh --port 27001 --eval "rs.initiate({_id:'rsConfig',configsvr:true,members:[{_id:0,host:'cfg1:27001',priority:3},{_id:1,host:'cfg2:27002'},{_id:2,host:'cfg3:27003'}]})"
docker exec -it cfg1 mongosh --port 27001 --eval "rs.status()"
PS D:\wh\mongo\hw_4> docker exec -it cfg1 mongosh --port 27001 --eval "rs.status()"
{
  set: 'rsConfig',
  date: ISODate('2025-10-16T16:56:04.219Z'),
  myState: 1,
  term: Long('1'),
  syncSourceHost: '',
  syncSourceId: -1,
  configsvr: true,
  heartbeatIntervalMillis: Long('2000'),
  majorityVoteCount: 2,
  writeMajorityCount: 2,
  votingMembersCount: 3,
  writableVotingMembersCount: 3,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1760633763, i: 1 }), t: Long('1') },
    lastCommittedWallTime: ISODate('2025-10-16T16:56:03.789Z'),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1760633763, i: 1 }), t: Long('1') },
    appliedOpTime: { ts: Timestamp({ t: 1760633763, i: 1 }), t: Long('1') },
    durableOpTime: { ts: Timestamp({ t: 1760633763, i: 1 }), t: Long('1') },
    writtenOpTime: { ts: Timestamp({ t: 1760633763, i: 1 }), t: Long('1') },
    lastAppliedWallTime: ISODate('2025-10-16T16:56:03.789Z'),
    lastDurableWallTime: ISODate('2025-10-16T16:56:03.789Z'),
    lastWrittenWallTime: ISODate('2025-10-16T16:56:03.789Z')
  },
  lastStableRecoveryTimestamp: Timestamp({ t: 1760633723, i: 1 }),
  electionCandidateMetrics: {
    lastElectionReason: 'electionTimeout',
    lastElectionDate: ISODate('2025-10-16T16:03:29.974Z'),
    electionTerm: Long('1'),
    lastCommittedOpTimeAtElection: { ts: Timestamp({ t: 1760630599, i: 1 }), t: Long('-1') },
    lastSeenWrittenOpTimeAtElection: { ts: Timestamp({ t: 1760630599, i: 1 }), t: Long('-1') },
    lastSeenOpTimeAtElection: { ts: Timestamp({ t: 1760630599, i: 1 }), t: Long('-1') },
    numVotesNeeded: 2,
    priorityAtElection: 3,
    electionTimeoutMillis: Long('10000'),
    numCatchUpOps: Long('0'),
    newTermStartDate: ISODate('2025-10-16T16:03:30.326Z'),
    wMajorityWriteAvailabilityDate: ISODate('2025-10-16T16:03:31.298Z')
  },
  members: [
    {
      _id: 0,
      name: 'cfg1:27001',
      health: 1,
      state: 1,
      stateStr: 'PRIMARY',
      uptime: 3183,
      optime: { ts: Timestamp({ t: 1760633763, i: 1 }), t: Long('1') },
      optimeDate: ISODate('2025-10-16T16:56:03.000Z'),
      optimeWritten: { ts: Timestamp({ t: 1760633763, i: 1 }), t: Long('1') },
      optimeWrittenDate: ISODate('2025-10-16T16:56:03.000Z'),
      lastAppliedWallTime: ISODate('2025-10-16T16:56:03.789Z'),
      lastDurableWallTime: ISODate('2025-10-16T16:56:03.789Z'),
      lastWrittenWallTime: ISODate('2025-10-16T16:56:03.789Z'),
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      electionTime: Timestamp({ t: 1760630609, i: 1 }),
      electionDate: ISODate('2025-10-16T16:03:29.000Z'),
      configVersion: 1,
      configTerm: 1,
      self: true,
      lastHeartbeatMessage: ''
    },
    {
      _id: 1,
      name: 'cfg2:27002',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 3164,
      optime: { ts: Timestamp({ t: 1760633763, i: 1 }), t: Long('1') },
      optimeDurable: { ts: Timestamp({ t: 1760633763, i: 1 }), t: Long('1') },
      optimeWritten: { ts: Timestamp({ t: 1760633763, i: 1 }), t: Long('1') },
      optimeDate: ISODate('2025-10-16T16:56:03.000Z'),
      optimeDurableDate: ISODate('2025-10-16T16:56:03.000Z'),
      optimeWrittenDate: ISODate('2025-10-16T16:56:03.000Z'),
      lastAppliedWallTime: ISODate('2025-10-16T16:56:03.789Z'),
      lastDurableWallTime: ISODate('2025-10-16T16:56:03.789Z'),
      lastWrittenWallTime: ISODate('2025-10-16T16:56:03.789Z'),
      lastHeartbeat: ISODate('2025-10-16T16:56:04.104Z'),
      lastHeartbeatRecv: ISODate('2025-10-16T16:56:02.855Z'),
      pingMs: Long('0'),
      lastHeartbeatMessage: '',
      syncSourceHost: 'cfg1:27001',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 1,
      configTerm: 1
    },
    {
      _id: 2,
      name: 'cfg3:27003',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 3164,
      optime: { ts: Timestamp({ t: 1760633763, i: 1 }), t: Long('1') },
      optimeDurable: { ts: Timestamp({ t: 1760633763, i: 1 }), t: Long('1') },
      optimeWritten: { ts: Timestamp({ t: 1760633763, i: 1 }), t: Long('1') },
      optimeDate: ISODate('2025-10-16T16:56:03.000Z'),
      optimeDurableDate: ISODate('2025-10-16T16:56:03.000Z'),
      optimeWrittenDate: ISODate('2025-10-16T16:56:03.000Z'),
      lastAppliedWallTime: ISODate('2025-10-16T16:56:03.789Z'),
      lastDurableWallTime: ISODate('2025-10-16T16:56:03.789Z'),
      lastWrittenWallTime: ISODate('2025-10-16T16:56:03.789Z'),
      lastHeartbeat: ISODate('2025-10-16T16:56:04.103Z'),
      lastHeartbeatRecv: ISODate('2025-10-16T16:56:03.038Z'),
      pingMs: Long('0'),
      lastHeartbeatMessage: '',
      syncSourceHost: 'cfg1:27001',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 1,
      configTerm: 1
    }
  ],
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1760633763, i: 1 }),
    signature: {
      hash: Binary.createFromBase64('AAAAAAAAAAAAAAAAAAAAAAAAAAA=', 0),
      keyId: Long('0')
    }
  },
  operationTime: Timestamp({ t: 1760633763, i: 1 })
}
~~~


## Создаём репликосет для 1 шарда
~~~sh
# RS1
docker exec -it sh1n1 mongosh --port 27011 --eval "rs.initiate({_id:'rsShard1',members:[{_id:0,host:'sh1n1:27011',priority:3},{_id:1,host:'sh1n2:27012',priority:2},{_id:2,host:'sh1n3:27013',priority:1}]})"
docker exec -it sh1n1 mongosh --port 27011 --eval "rs.status()"

PS D:\wh\mongo\hw_4> docker exec -it sh1n1 mongosh --port 27011 --eval "rs.status()"
{
  set: 'rsShard1',
  date: ISODate('2025-10-16T16:05:29.160Z'),
  myState: 1,
  term: Long('1'),
  syncSourceHost: '',
  syncSourceId: -1,
  heartbeatIntervalMillis: Long('2000'),
  majorityVoteCount: 2,
  writeMajorityCount: 2,
  votingMembersCount: 3,
  writableVotingMembersCount: 3,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1760630728, i: 1 }), t: Long('1') },
    lastCommittedWallTime: ISODate('2025-10-16T16:05:28.386Z'),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1760630728, i: 1 }), t: Long('1') },
    appliedOpTime: { ts: Timestamp({ t: 1760630728, i: 1 }), t: Long('1') },
    durableOpTime: { ts: Timestamp({ t: 1760630728, i: 1 }), t: Long('1') },
    writtenOpTime: { ts: Timestamp({ t: 1760630728, i: 1 }), t: Long('1') },
    lastAppliedWallTime: ISODate('2025-10-16T16:05:28.386Z'),
    lastDurableWallTime: ISODate('2025-10-16T16:05:28.386Z'),
    lastWrittenWallTime: ISODate('2025-10-16T16:05:28.386Z')
  },
  lastStableRecoveryTimestamp: Timestamp({ t: 1760630713, i: 1 }),
  electionCandidateMetrics: {
    lastElectionReason: 'electionTimeout',
    lastElectionDate: ISODate('2025-10-16T16:05:24.592Z'),
    electionTerm: Long('1'),
    lastCommittedOpTimeAtElection: { ts: Timestamp({ t: 1760630713, i: 1 }), t: Long('-1') },
    lastSeenWrittenOpTimeAtElection: { ts: Timestamp({ t: 1760630713, i: 1 }), t: Long('-1') },
    lastSeenOpTimeAtElection: { ts: Timestamp({ t: 1760630713, i: 1 }), t: Long('-1') },
    numVotesNeeded: 2,
    priorityAtElection: 3,
    electionTimeoutMillis: Long('10000'),
    numCatchUpOps: Long('0'),
    newTermStartDate: ISODate('2025-10-16T16:05:25.314Z'),
    wMajorityWriteAvailabilityDate: ISODate('2025-10-16T16:05:25.996Z')
  },
  members: [
    {
      _id: 0,
      name: 'sh1n1:27011',
      health: 1,
      state: 1,
      stateStr: 'PRIMARY',
      uptime: 148,
      optime: { ts: Timestamp({ t: 1760630728, i: 1 }), t: Long('1') },
      optimeDate: ISODate('2025-10-16T16:05:28.000Z'),
      optimeWritten: { ts: Timestamp({ t: 1760630728, i: 1 }), t: Long('1') },
      optimeWrittenDate: ISODate('2025-10-16T16:05:28.000Z'),
      lastAppliedWallTime: ISODate('2025-10-16T16:05:28.386Z'),
      lastDurableWallTime: ISODate('2025-10-16T16:05:28.386Z'),
      lastWrittenWallTime: ISODate('2025-10-16T16:05:28.386Z'),
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: 'Could not find member to sync from',
      electionTime: Timestamp({ t: 1760630724, i: 1 }),
      electionDate: ISODate('2025-10-16T16:05:24.000Z'),
      configVersion: 1,
      configTerm: 1,
      self: true,
      lastHeartbeatMessage: ''
    },
    {
      _id: 1,
      name: 'sh1n2:27012',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 15,
      optime: { ts: Timestamp({ t: 1760630725, i: 2 }), t: Long('1') },
      optimeDurable: { ts: Timestamp({ t: 1760630727, i: 1 }), t: Long('1') },
      optimeWritten: { ts: Timestamp({ t: 1760630728, i: 1 }), t: Long('1') },
      optimeDate: ISODate('2025-10-16T16:05:25.000Z'),
      optimeDurableDate: ISODate('2025-10-16T16:05:27.000Z'),
      optimeWrittenDate: ISODate('2025-10-16T16:05:28.000Z'),
      lastAppliedWallTime: ISODate('2025-10-16T16:05:25.314Z'),
      lastDurableWallTime: ISODate('2025-10-16T16:05:28.386Z'),
      lastWrittenWallTime: ISODate('2025-10-16T16:05:28.386Z'),
      lastHeartbeat: ISODate('2025-10-16T16:05:28.796Z'),
      lastHeartbeatRecv: ISODate('2025-10-16T16:05:27.794Z'),
      pingMs: Long('0'),
      lastHeartbeatMessage: '',
      syncSourceHost: 'sh1n1:27011',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 1,
      configTerm: 1
    },
    {
      _id: 2,
      name: 'sh1n3:27013',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 15,
      optime: { ts: Timestamp({ t: 1760630725, i: 2 }), t: Long('1') },
      optimeDurable: { ts: Timestamp({ t: 1760630727, i: 1 }), t: Long('1') },
      optimeWritten: { ts: Timestamp({ t: 1760630728, i: 1 }), t: Long('1') },
      optimeDate: ISODate('2025-10-16T16:05:25.000Z'),
      optimeDurableDate: ISODate('2025-10-16T16:05:27.000Z'),
      optimeWrittenDate: ISODate('2025-10-16T16:05:28.000Z'),
      lastAppliedWallTime: ISODate('2025-10-16T16:05:25.544Z'),
      lastDurableWallTime: ISODate('2025-10-16T16:05:28.386Z'),
      lastWrittenWallTime: ISODate('2025-10-16T16:05:28.386Z'),
      lastHeartbeat: ISODate('2025-10-16T16:05:28.799Z'),
      lastHeartbeatRecv: ISODate('2025-10-16T16:05:27.799Z'),
      pingMs: Long('0'),
      lastHeartbeatMessage: '',
      syncSourceHost: 'sh1n1:27011',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 1,
      configTerm: 1
    }
  ],
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1760630728, i: 2 }),
    signature: {
      hash: Binary.createFromBase64('AAAAAAAAAAAAAAAAAAAAAAAAAAA=', 0),
      keyId: Long('0')
    }
  },
  operationTime: Timestamp({ t: 1760630728, i: 1 })
}
~~~

## Создаём репликосет для 2 шарда
~~~sh
# RS2
docker exec -it sh2n1 mongosh --port 27021 --eval "rs.initiate({_id:'rsShard2',members:[{_id:0,host:'sh2n1:27021',priority:3},{_id:1,host:'sh2n2:27022',priority:2},{_id:2,host:'sh2n3:27023',priority:1}]})"
docker exec -it sh2n1 mongosh --port 27021 --eval "rs.status()"

PS D:\wh\mongo\hw_4> docker exec -it sh2n1 mongosh --port 27021 --eval "rs.status()"
{
  set: 'rsShard2',
  date: ISODate('2025-10-16T16:06:19.375Z'),
  myState: 1,
  term: Long('1'),
  syncSourceHost: '',
  syncSourceId: -1,
  heartbeatIntervalMillis: Long('2000'),
  majorityVoteCount: 2,
  writeMajorityCount: 2,
  votingMembersCount: 3,
  writableVotingMembersCount: 3,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1760630769, i: 1 }), t: Long('1') },
    lastCommittedWallTime: ISODate('2025-10-16T16:06:09.636Z'),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1760630769, i: 1 }), t: Long('1') },
    appliedOpTime: { ts: Timestamp({ t: 1760630769, i: 1 }), t: Long('1') },
    durableOpTime: { ts: Timestamp({ t: 1760630769, i: 1 }), t: Long('1') },
    writtenOpTime: { ts: Timestamp({ t: 1760630769, i: 1 }), t: Long('1') },
    lastAppliedWallTime: ISODate('2025-10-16T16:06:09.636Z'),
    lastDurableWallTime: ISODate('2025-10-16T16:06:09.636Z'),
    lastWrittenWallTime: ISODate('2025-10-16T16:06:09.636Z')
  },
  lastStableRecoveryTimestamp: Timestamp({ t: 1760630752, i: 1 }),
  electionCandidateMetrics: {
    lastElectionReason: 'electionTimeout',
    lastElectionDate: ISODate('2025-10-16T16:06:04.524Z'),
    electionTerm: Long('1'),
    lastCommittedOpTimeAtElection: { ts: Timestamp({ t: 1760630752, i: 1 }), t: Long('-1') },
    lastSeenWrittenOpTimeAtElection: { ts: Timestamp({ t: 1760630752, i: 1 }), t: Long('-1') },
    lastSeenOpTimeAtElection: { ts: Timestamp({ t: 1760630752, i: 1 }), t: Long('-1') },
    numVotesNeeded: 2,
    priorityAtElection: 3,
    electionTimeoutMillis: Long('10000'),
    numCatchUpOps: Long('0'),
    newTermStartDate: ISODate('2025-10-16T16:06:05.322Z'),
    wMajorityWriteAvailabilityDate: ISODate('2025-10-16T16:06:06.347Z')
  },
  members: [
    {
      _id: 0,
      name: 'sh2n1:27021',
      health: 1,
      state: 1,
      stateStr: 'PRIMARY',
      uptime: 199,
      optime: { ts: Timestamp({ t: 1760630769, i: 1 }), t: Long('1') },
      optimeDate: ISODate('2025-10-16T16:06:09.000Z'),
      optimeWritten: { ts: Timestamp({ t: 1760630769, i: 1 }), t: Long('1') },
      optimeWrittenDate: ISODate('2025-10-16T16:06:09.000Z'),
      lastAppliedWallTime: ISODate('2025-10-16T16:06:09.636Z'),
      lastDurableWallTime: ISODate('2025-10-16T16:06:09.636Z'),
      lastWrittenWallTime: ISODate('2025-10-16T16:06:09.636Z'),
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: 'Could not find member to sync from',
      electionTime: Timestamp({ t: 1760630764, i: 1 }),
      electionDate: ISODate('2025-10-16T16:06:04.000Z'),
      configVersion: 1,
      configTerm: 1,
      self: true,
      lastHeartbeatMessage: ''
    },
    {
      _id: 1,
      name: 'sh2n2:27022',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 26,
      optime: { ts: Timestamp({ t: 1760630769, i: 1 }), t: Long('1') },
      optimeDurable: { ts: Timestamp({ t: 1760630769, i: 1 }), t: Long('1') },
      optimeWritten: { ts: Timestamp({ t: 1760630769, i: 1 }), t: Long('1') },
      optimeDate: ISODate('2025-10-16T16:06:09.000Z'),
      optimeDurableDate: ISODate('2025-10-16T16:06:09.000Z'),
      optimeWrittenDate: ISODate('2025-10-16T16:06:09.000Z'),
      lastAppliedWallTime: ISODate('2025-10-16T16:06:09.636Z'),
      lastDurableWallTime: ISODate('2025-10-16T16:06:09.636Z'),
      lastWrittenWallTime: ISODate('2025-10-16T16:06:09.636Z'),
      lastHeartbeat: ISODate('2025-10-16T16:06:18.745Z'),
      lastHeartbeatRecv: ISODate('2025-10-16T16:06:19.245Z'),
      pingMs: Long('0'),
      lastHeartbeatMessage: '',
      syncSourceHost: 'sh2n1:27021',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 1,
      configTerm: 1
    },
    {
      _id: 2,
      name: 'sh2n3:27023',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 26,
      optime: { ts: Timestamp({ t: 1760630769, i: 1 }), t: Long('1') },
      optimeDurable: { ts: Timestamp({ t: 1760630769, i: 1 }), t: Long('1') },
      optimeWritten: { ts: Timestamp({ t: 1760630769, i: 1 }), t: Long('1') },
      optimeDate: ISODate('2025-10-16T16:06:09.000Z'),
      optimeDurableDate: ISODate('2025-10-16T16:06:09.000Z'),
      optimeWrittenDate: ISODate('2025-10-16T16:06:09.000Z'),
      lastAppliedWallTime: ISODate('2025-10-16T16:06:09.636Z'),
      lastDurableWallTime: ISODate('2025-10-16T16:06:09.636Z'),
      lastWrittenWallTime: ISODate('2025-10-16T16:06:09.636Z'),
      lastHeartbeat: ISODate('2025-10-16T16:06:18.752Z'),
      lastHeartbeatRecv: ISODate('2025-10-16T16:06:17.754Z'),
      pingMs: Long('0'),
      lastHeartbeatMessage: '',
      syncSourceHost: 'sh2n1:27021',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 1,
      configTerm: 1
    }
  ],
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1760630769, i: 1 }),
    signature: {
      hash: Binary.createFromBase64('AAAAAAAAAAAAAAAAAAAAAAAAAAA=', 0),
      keyId: Long('0')
    }
  },
  operationTime: Timestamp({ t: 1760630769, i: 1 })
}
~~~

## Рестартим mongos
~~~sh
docker compose restart mongos1 mongos2
~~~


## Добавляем шарды
~~~sh
docker exec -it mongos1 mongosh --port 27000 --eval "sh.addShard('rsShard1/sh1n1:27011,sh1n2:27012,sh1n3:27013'); sh.addShard('rsShard2/sh2n1:27021,sh2n2:27022,sh2n3:27023')" 
PS D:\wh\mongo\hw_4> docker exec -it mongos1 mongosh --port 27000 --eval "sh.status()"                                                                                                                          
shardingVersion
{ _id: 1, clusterId: ObjectId('68f117537d5a05a0e6ad6827') }
---
shards
[
  {
    _id: 'rsShard1',
    host: 'rsShard1/sh1n1:27011,sh1n2:27012,sh1n3:27013',
    state: 1,
    topologyTime: Timestamp({ t: 1760630877, i: 9 }),
    replSetConfigVersion: Long('-1')
  },
  {
    _id: 'rsShard2',
    host: 'rsShard2/sh2n1:27021,sh2n2:27022,sh2n3:27023',
    state: 1,
    topologyTime: Timestamp({ t: 1760630879, i: 8 }),
    replSetConfigVersion: Long('-1')
  }
]
---
active mongoses
[ { '8.0.14': 2 } ]
---
autosplit
{ 'Currently enabled': 'yes' }
---
balancer
{
  'Currently enabled': 'yes',
  'Currently running': 'no',
  'Failed balancer rounds in last 5 attempts': 0,
  'Migration Results for the last 24 hours': 'No recent migrations'
}
---
shardedDataDistribution
[
  {
    ns: 'config.system.sessions',
    shards: [
      {
        shardName: 'rsShard1',
        numOrphanedDocs: 0,
        numOwnedDocuments: 8,
        ownedSizeBytes: 792,
        orphanedSizeBytes: 0
      }
    ]
  }
]
---
databases
[
  {
    database: { _id: 'config', primary: 'config', partitioned: true },
    collections: {
      'config.system.sessions': {
        shardKey: { _id: 1 },
        unique: false,
        balancing: true,
        chunkMetadata: [ { shard: 'rsShard1', nChunks: 1 } ],
        chunks: [
          { min: { _id: MinKey() }, max: { _id: MaxKey() }, 'on shard': 'rsShard1', 'last modified': Timestamp({ t: 1, i: 0 }) }
        ],
        tags: []
      }
    }
  }
]
~~~

##  Загружаем данные
~~~sh
New-Item -ItemType Directory -Force -Path .\mongo_data\datasets | Out-Null
curl.exe -L "https://dl.dropboxusercontent.com/s/p75zp1karqg6nnn/stocks.zip" -o .\mongo_data\datasets\stocks.zip


Expand-Archive -Path .\mongo_data\datasets\stocks.zip -DestinationPath .\mongo_data\datasets\stocks -Force

$dumpDir = (Resolve-Path ".\mongo_data\datasets\stocks\dump").Path
# Загрузка дампа
docker run --rm --network mongo-net -v "${dumpDir}:/dump:ro" mongo:8.0.14 `
  mongorestore --uri "mongodb://mongos1:27000" `
  --numInsertionWorkersPerCollection 8 `
  /dump


    2025-10-16T17:52:04.975+0000    finished restoring stocks.values (4308303 documents, 0 failures)
    2025-10-16T17:52:04.976+0000    no indexes to restore for collection stocks.values
    2025-10-16T17:52:04.976+0000    4308303 document(s) restored successfully. 0 document(s) failed to restore.
~~~

##  Подключимся к mongos1
~~~sh
docker exec -it mongos1 mongosh --port 27000
~~~


## Для шардирования можно использовать stock_symbol, он равномерно распределит данные по акциям + запросы часто затрагивают конкретную акцию или выборку. Также неплохо может быть date (равномерно распределено, + по дате частое обращение), но будут проблемы при добавлении данных. Попробуем шардировать по stock_symbol + date

~~~javascript
// включаем шардирование базы
sh.enableSharding("stocks")
// создаём индекс
use stocks
db.values.createIndex({ stock_symbol: 1, date: 1 })
// шардируем коллекцию
sh.shardCollection("stocks.values", { stock_symbol: 1, date: 1 })
sh.status()

chunks: [
  { min: { stock_symbol: MinKey(), date: MinKey() }, max: { stock_symbol: 'ATRO', date: '2007-11-28' }, 'on shard': 'rsShard1', 'last modified': Timestamp({ t: 2, i: 0 }) },
  { min: { stock_symbol: 'ATRO', date: '2007-11-28' }, max: { stock_symbol: 'CALM', date: '2003-04-22' }, 'on shard': 'rsShard1', 'last modified': Timestamp({ t: 3, i: 0 }) },
  { min: { stock_symbol: 'CALM', date: '2003-04-22' }, max: { stock_symbol: MaxKey(), date: MaxKey() }, 'on shard': 'rsShard2', 'last modified': Timestamp({ t: 3, i: 1 }) }
],

// rShard1 — 2 чанка, rsShard2 — 1 чанк
// rsShard1: owned  1.54 млн доков (~268 МБ)
// rsShard2: owned 2.77 млн доков (~481 МБ)
~~~


## MapReduce & Aggregation 
~~~javascript
// MapReduce
var mapStock = function () {
  if (this.open != null && this.close != null) {
    var d = this.open - this.close;
    if (d > 100) emit(this.stock_symbol, { count: 1, sum: d });
  }
};
var reduceStock = function (k, vs) {
  var out = { count: 0, sum: 0 };
  for (var i = 0; i < vs.length; i++) {
    out.count += vs[i].count || 0;
    out.sum   += vs[i].sum   || 0;
  }
  return out;
};

var tMR1 = Date.now();
var mrInline = db.values.mapReduce(mapStock, reduceStock, { out: { inline: 1 } });
var ms = Date.now() - tMR1;
({ ms, sample: mrInline.results.slice(0, 10) })


// {
//   mrTimeInlineMs: 63704,
//   sample: [
//     { _id: 'MCHXP', value: { count: 1, sum: 106.72999999999999 } },
//     { _id: 'ATCO', value: { count: 1, sum: 30997.62 } },
//     { _id: 'FSCI', value: { count: 1, sum: 15643 } },
//     { _id: 'AMPH', value: { count: 1, sum: 8083.19 } },
//     { _id: 'GILT', value: { count: 1, sum: 2399.37 } },
//     { _id: 'MIND', value: { count: 1, sum: 2313.44 } },
//     { _id: 'BORD', value: { count: 8, sum: 3250 } },
//     { _id: 'NMSS', value: { count: 1, sum: 1435 } },
//     { _id: 'BLDP', value: { count: 2, sum: 25836.24 } },
//     { _id: 'BNHN', value: { count: 2, sum: 16387.5 } }
//   ]
// }

// С предфильтрацией
var q = { $expr: { $gt: [ { $subtract: ["$open", "$close"] }, 100 ] } };
var t = Date.now();
var mr = db.values.mapReduce(mapStock, reduceStock, { query: q, out: { inline: 1 } });
var ms = Date.now() - t;
({ ms, sample: mr.results.slice(0, 10) })

// {
//   ms: 8886,
//   sample: [
//     { _id: 'NMSS', value: { count: 1, sum: 1435 } },
//     { _id: 'ATCO', value: { count: 1, sum: 30997.62 } },
//     { _id: 'FSCI', value: { count: 1, sum: 15643 } },
//     { _id: 'NXXI', value: { count: 1, sum: 2142.75 } },
//     { _id: 'AMPH', value: { count: 1, sum: 8083.19 } },
//     { _id: 'BORD', value: { count: 8, sum: 3250 } },
//     { _id: 'MIND', value: { count: 1, sum: 2313.44 } },
//     { _id: 'BLDP', value: { count: 2, sum: 25836.24 } },
//     { _id: 'BNHN', value: { count: 2, sum: 16387.5 } },
//     { _id: 'AWRE', value: { count: 1, sum: 29543.75 } }
//   ]
// }


// Aggregation
var tAgg = Date.now();
var aggRes = db.values.aggregate([
  { $match: { $expr: { $gt: [ { $subtract: ["$open", "$close"] }, 100 ] } } },
  { $group: {
      _id: "$stock_symbol",
      count: { $sum: 1 },
      sum:   { $sum: { $subtract: ["$open", "$close"] } }
  } },
  { $sort: { count: -1 } }
]).toArray()
var ms = Date.now() - tAgg;
({ ms, sample: aggRes.slice(0, 10) })

// {
//   ms: 3720,
//   sample: [
//     { _id: 'BORD', count: 8, sum: 3250 },
//     { _id: 'BNHN', count: 2, sum: 16387.5 },
//     { _id: 'BLDP', count: 2, sum: 25836.24 },
//     { _id: 'MCHXP', count: 1, sum: 106.72999999999999 },
//     { _id: 'AWRE', count: 1, sum: 29543.75 },
//     { _id: 'ATCO', count: 1, sum: 30997.62 },
//     { _id: 'AMPH', count: 1, sum: 8083.19 },
//     { _id: 'NMSS', count: 1, sum: 1435 },
//     { _id: 'MIND', count: 1, sum: 2313.44 },
//     { _id: 'GILT', count: 1, sum: 2399.37 }
//   ]
// }

// Aggregation показал себя быстрее MapReduce
~~~


## MapReduce & Aggregation по полям шардирования
~~~javascript
var symbol = "BORD";
var from   = "2000-01-01";
var to     = "2000-12-31";

var qTarget = {
  stock_symbol: symbol,
  date: { $gte: from, $lte: to },
  $expr: { $gt: [ { $subtract: ["$open","$close"] }, 100 ] }
};

var mapStock = function () {
  if (this.open != null && this.close != null) {
    var d = this.open - this.close;
    if (d > 100) emit(this.stock_symbol, { count: 1, sum: d });
  }
};
var reduceStock = function (k, vs) {
  var out = { count: 0, sum: 0 };
  for (var i = 0; i < vs.length; i++) { out.count += vs[i].count || 0; out.sum += vs[i].sum || 0; }
  return out;
};

var tMR = Date.now();
var mr = db.values.mapReduce(mapStock, reduceStock, { query: qTarget, out: { inline: 1 } });
var mrMs = Date.now() - tMR;

var mrFlat = mr.results
  .map(r => ({ _id: r._id, count: r.value.count, sum: r.value.sum }))
  .sort((a,b) => (b.count - a.count) || (b.sum - a.sum));

var tAgg = Date.now();
var agg = db.values.aggregate([
  { $match: { stock_symbol: symbol, date: { $gte: from, $lte: to } } },
  { $set:   { diff: { $subtract: ["$open","$close"] } } },
  { $match: { diff: { $gt: 100 } } },
  { $group: { _id: "$stock_symbol", count: { $sum: 1 }, sum: { $sum: "$diff" } } },
  { $sort:  { count: -1, sum: -1 } }
], { allowDiskUse: true }).toArray();
var aggMs = Date.now() - tAgg;


({ mrMs, aggMs, mrSample: mrFlat.slice(0,10), aggSample: agg.slice(0,10) })

// {
//   mrMs: 120,
//   aggMs: 22,
//   mrSample: [ { _id: 'BORD', count: 8, sum: 3250 } ],
//   aggSample: [ { _id: 'BORD', count: 8, sum: 3250 } ]
// }


// Aggregation также быстрее MapReduce
~~~
