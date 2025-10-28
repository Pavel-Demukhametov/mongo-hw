# ДЗ 5

ФИО: Демухаметов Павел

## Окружение
- Windows 11 + Docker
- Образ: mongo:8.0.14


## Настройка аутентификации
~~~sh
docker run -d `
  --name mongodb-server `
  --network mongo-network `
  --restart unless-stopped `
  -v "D:\wh\mongo\hw_5\data:/data/db" `
  mongo:latest

docker run -it --rm `
  --name mongo-client `
  --network mongo-network `
  mongo:latest `
  mongosh --host mongodb-server
~~~

~~~javascript
// Создаем администратора
use admin
db.createUser({
  user: "adminUser",
  pwd: "12345",
  roles: [
    { role: "root", db: "admin" }
  ]
})

// Создаем пользователя для работы с данными
db.createUser({
  user: "dataUser", 
  pwd: "54321",
  roles: [
    { role: "readWrite", db: "testdb" }
  ]
})

exit
~~~

~~~sh
docker stop mongodb-server
docker rm mongodb-server

# Запускаем с включенной аутентификацией
docker run -d `
  --name mongodb-server `
  --network mongo-network `
  --restart unless-stopped `
  -v "D:\wh\mongo\hw_5\data:/data/db" `
  mongo:latest `
  mongod --auth


# Подключаемся с ролью adminUser
docker run -it --rm `
  --network mongo-network `
  mongo:latest `
  mongosh --host mongodb-server -u adminUser -p 12345 --authenticationDatabase admin

use admin
# Успешное выполнение (role: "root")
db.getUsers()
~~~

~~~sh
# Подключаемся с созданной ролью dataUser
docker run -it --rm `
  --network mongo-network `
  mongo:latest `
  mongosh --host mongodb-server -u dataUser -p 54321 --authenticationDatabase admin

use admin
# Ошибка (не доступа к admin, role: "readWrite")
db.getUsers()
MongoServerError[Unauthorized]: not authorized on admin to execute command { usersInfo: 1, lsid: { id: UUID("7c2aad66-ca0e-4ee9-a955-32b5bfda4160") }, $db: "admin" }

# Подключение с неверным паролем
docker run -it --rm `
  --network mongo-network `
  mongo:latest `
  mongosh --host mongodb-server -u adminUser -p wrong --authenticationDatabase admin

Connecting to:          mongodb://<credentials>@mongodb-server:27017/?directConnection=true&authSource=admin&appName=mongosh+2.5.8
MongoServerError: Authentication failed.
~~~

## Настройка валидации
~~~javascript
use testdb

const currentYear = new Date().getFullYear();
// добавление валидаторов 
db.createCollection("users", {
   validator: {
      $jsonSchema: {
         bsonType: "object",
         title: "User Object Validation",
         required: [ "name", "email", "birth_year" ],
         properties: {
            name: {
               bsonType: "string",
               description: "must be a string between 2 and 50 characters and is required",
               minLength: 2,
               maxLength: 50
            },
            email: {
               bsonType: "string",
               pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$",
               description: "must be a valid email address and is required"
            },
            birth_year: {
               bsonType: "int",
               minimum: 1900,
               maximum: currentYear,
               description: "must be an integer between 1900 and the current year"
            }
         }
      }
   },
})

db.getCollectionInfos({ name: "users" })
testdb> db.getCollectionInfos({ name: "users" })
[
  {
    name: 'users',
    type: 'collection',
    options: {
      validator: {
        '$jsonSchema': {
          bsonType: 'object',
          title: 'User Object Validation',
          required: [ 'name', 'email', 'birth_year' ],
          properties: {
            name: {
              bsonType: 'string',
              description: 'must be a string between 2 and 50 characters and is required',
              minLength: 2,
              maxLength: 50
            },
            email: {
              bsonType: 'string',
              pattern: '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$',
              description: 'must be a valid email address and is required'
            },
            birth_year: {
              bsonType: 'int',
              minimum: 1900,
              maximum: 2025,
              description: 'must be an integer between 1900 and the current year'
            }
          }
        }
      }
    },
    info: {
      readOnly: false,
      uuid: UUID('d4871407-f07e-4b3b-9e49-052c5cf41ba6')
    },
    idIndex: { v: 2, key: { _id: 1 }, name: '_id_' }
  }
]
~~~

~~~javascript
// Успешная вставка
db.users.insertOne({
  name: "LI",
  email: "Li@example.com",
  birth_year: 2000
})

{
  acknowledged: true,
  insertedId: ObjectId('6900bc8b5992e0df31ce5f47')
}


// Ошибка с валидацией по длине
db.users.insertOne({
  name: "L",
  email: "Li@example.com",
  birth_year: 2000
})
Uncaught:
MongoServerError: Document failed validation
Additional information: {
  failingDocumentId: ObjectId('6900bcb75992e0df31ce5f48'),
  details: {
    operatorName: '$jsonSchema',
    title: 'User Object Validation',
    schemaRulesNotSatisfied: [
      {
        operatorName: 'properties',
        propertiesNotSatisfied: [
          {
            propertyName: 'name',
            description: 'must be a string between 2 and 50 characters and is required',
            details: [
              {
                operatorName: 'minLength',
                specifiedAs: { minLength: 2 },
                reason: 'specified string length was not satisfied',
                consideredValue: 'L'
              }
            ]
          }
        ]
      }
    ]
  }
}


// Ошибка с регулярным выражением
db.users.insertOne({
  name: "Li",
  email: "Li@example,com",
  birth_year: 2000
})

Uncaught:
MongoServerError: Document failed validation
Additional information: {
  failingDocumentId: ObjectId('6900bcef5992e0df31ce5f4a'),
  details: {
    operatorName: '$jsonSchema',
    title: 'User Object Validation',
    schemaRulesNotSatisfied: [
      {
        operatorName: 'properties',
        propertiesNotSatisfied: [
          {
            propertyName: 'email',
            description: 'must be a valid email address and is required',
            details: [
              {
                operatorName: 'pattern',
                specifiedAs: {
                  pattern: '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$'
                },
                reason: 'regular expression did not match',
                consideredValue: 'Li@example,com'
              }
            ]
          }
        ]
      }
    ]
  }
}


// Ошибка с обязательным полем
db.users.insertOne({
  name: "Li",
  email: "Li@example.com",
})

MongoServerError: Document failed validation
Additional information: {
  failingDocumentId: ObjectId('6900bd155992e0df31ce5f4b'),
  details: {
    operatorName: '$jsonSchema',
    title: 'User Object Validation',
    schemaRulesNotSatisfied: [
      {
        operatorName: 'required',
        specifiedAs: { required: [ 'name', 'email', 'birth_year' ] },
        missingProperties: [ 'birth_year' ]
      }
    ]
  }
}
~~~


## Экспорт/импорт данных
Использовался датасет https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_hour.geojson
~~~sh
# используем adminUser (dataUser не сможет добавить валидацию) 
docker exec mongodb-server mongoimport `
  -u adminUser  -p 12345 --authenticationDatabase admin `
  --db testdb `
  --collection earthquakes `
  --file /data/db/earthquakes.json `
  --jsonArray

# Успешный импорт
2025-10-28T13:36:45.642+0000    connected to: mongodb://localhost/
2025-10-28T13:36:45.943+0000    5 document(s) imported successfully. 0 document(s) failed to import.


docker run -it --rm `
  --network mongo-network `
  mongo:latest `
  mongosh --host mongodb-server -u adminUser -p 12345 --authenticationDatabase admin

use testdb

# Проверка импорта
db.earthquakes.find()
testdb> db.earthquakes.find()
[
  {
    _id: ObjectId('6900c6ed091da765d36b8e03'),
    type: 'Feature',
    properties: {
      mag: 1.3,
      place: '57 km SSW of Manley Hot Springs, Alaska',
      time: Long('1761657359338'),
      updated: Long('1761657493346'),
      tz: null,
      url: 'https://earthquake.usgs.gov/earthquakes/eventpage/ak025du0lftp',
      detail: 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/detail/ak025du0lftp.geojson',
      felt: null,
      cdi: null,
      mmi: null,
      alert: null,
      status: 'automatic',
      tsunami: 0,
      sig: 26,
      net: 'ak',
      code: '025du0lftp',
      ids: ',ak025du0lftp,',
      sources: ',ak,',
      types: ',origin,phase-data,',
      nst: null,
      dmin: null,
      rms: 0.27,
      gap: null,
      magType: 'ml',
      type: 'earthquake',
      title: 'M 1.3 - 57 km SSW of Manley Hot Springs, Alaska'
    },
    geometry: { type: 'Point', coordinates: [ -151.2758, 64.567, 3 ] },
    id: 'ak025du0lftp'
  },
  {
    _id: ObjectId('6900c6ed091da765d36b8e04'),
    type: 'Feature',
    properties: {
      mag: 2.2163283824920654,
      place: '5 km W of Gorst, Washington',
      time: Long('1761656094810'),
      updated: Long('1761656306840'),
      tz: null,
      url: 'https://earthquake.usgs.gov/earthquakes/eventpage/uw62208526',
      detail: 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/detail/uw62208526.geojson',
      felt: null,
      cdi: null,
      mmi: null,
      alert: null,
      status: 'automatic',
      tsunami: 0,
      sig: 76,
      net: 'uw',
      code: '62208526',
      ids: ',uw62208526,',
      sources: ',uw,',
      types: ',origin,phase-data,',
      nst: 37,
      dmin: null,
      rms: 0.109999999,
      gap: 37,
      magType: 'ml',
      type: 'earthquake',
      title: 'M 2.2 - 5 km W of Gorst, Washington'
    },
    geometry: {
      type: 'Point',
      coordinates: [ -122.77149963378906, 47.52483367919922, 23.0200004577637 ]
    },
    id: 'uw62208526'
  },
  {
    _id: ObjectId('6900c6ed091da765d36b8e05'),
    type: 'Feature',
    properties: {
      mag: 0.72,
      place: '7 km WNW of Cobb, CA',
      time: Long('1761654904820'),
      updated: Long('1761657137704'),
      tz: null,
      url: 'https://earthquake.usgs.gov/earthquakes/eventpage/nc75255686',
      detail: 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/detail/nc75255686.geojson',
      felt: null,
      cdi: null,
      mmi: null,
      alert: null,
      status: 'automatic',
      tsunami: 0,
      sig: 8,
      net: 'nc',
      code: '75255686',
      ids: ',nc75255686,',
      sources: ',nc,',
      types: ',nearby-cities,origin,phase-data,scitech-link,',
      nst: 11,
      dmin: 0.006396,
      rms: 0.01,
      gap: 71,
      magType: 'md',
      type: 'earthquake',
      title: 'M 0.7 - 7 km WNW of Cobb, CA'
    },
    geometry: {
      type: 'Point',
      coordinates: [ -122.798500061035, 38.837833404541, 1.71000003814697 ]
    },
    id: 'nc75255686'
  },
  {
    _id: ObjectId('6900c6ed091da765d36b8e06'),
    type: 'Feature',
    properties: {
      mag: 2.74,
      place: '23 km SSW of Mammoth, Wyoming',
      time: Long('1761656443710'),
      updated: Long('1761657632010'),
      tz: null,
      url: 'https://earthquake.usgs.gov/earthquakes/eventpage/uu80120471',
      detail: 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/detail/uu80120471.geojson',
      felt: null,
      cdi: null,
      mmi: null,
      alert: null,
      status: 'reviewed',
      tsunami: 0,
      sig: 116,
      net: 'uu',
      code: '80120471',
      ids: ',uu80120471,',
      sources: ',uu,',
      types: ',origin,phase-data,',
      nst: 34,
      dmin: 0.01072,
      rms: 0.21,
      gap: 48,
      magType: 'ml',
      type: 'earthquake',
      title: 'M 2.7 - 23 km SSW of Mammoth, Wyoming'
    },
    geometry: { type: 'Point', coordinates: [ -110.852, 44.799, 5.64 ] },
    id: 'uu80120471'
  },
  {
    _id: ObjectId('6900c6ed091da765d36b8e07'),
    type: 'Feature',
    properties: {
      mag: 1.46,
      place: '9 km NW of Santa Paula, CA',
      time: Long('1761655192140'),
      updated: Long('1761657582164'),
      tz: null,
      url: 'https://earthquake.usgs.gov/earthquakes/eventpage/ci41321584',
      detail: 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/detail/ci41321584.geojson',
      felt: null,
      cdi: null,
      mmi: null,
      alert: null,
      status: 'reviewed',
      tsunami: 0,
      sig: 33,
      net: 'ci',
      code: '41321584',
      ids: ',ci41321584,',
      sources: ',ci,',
      types: ',nearby-cities,origin,phase-data,scitech-link,',
      nst: 33,
      dmin: 0.02088,
      rms: 0.3,
      gap: 50,
      magType: 'ml',
      type: 'earthquake',
      title: 'M 1.5 - 9 km NW of Santa Paula, CA'
    },
    geometry: { type: 'Point', coordinates: [ -119.1218333, 34.4205, 14.26 ] },
    id: 'ci41321584'
  }
]
~~~
~~~javascript

// Добавим валидацию
db.runCommand({
   collMod: "earthquakes",
   validator: {
      $jsonSchema: {
         bsonType: "object",
         required: [ "id" ],
         properties: {
            properties: {
               bsonType: "object",
               properties: {
                  mag: { 
                     bsonType: "number",
                     minimum: 0,
                     maximum: 10,
                     description: "Magnitude must be a number between 0 and 10"
                  }
               }
            }
         }
      }
   },
   validationAction: "error"
})
{ ok: 1 }

// Успешное добавление
db.earthquakes.insertOne({
  type: 'Feature',
  properties: { mag: 10, place: 'Test', time: new Date() },
  geometry: { type: 'Point', coordinates: [0, 0, 0] },
  id: 'test001'
})
{
  acknowledged: true,
  insertedId: ObjectId('6900cc3fec4c1337fbce5f48')
}

// Ошибка валидации (mag > 10)
db.earthquakes.insertOne({
  type: 'Feature',
  properties: { mag: 11, place: 'Test', time: new Date() },
  geometry: { type: 'Point', coordinates: [0, 0, 0] },
  id: 'test001'
})
MongoServerError: Document failed validation...
~~~

~~~sh
# Бекап testdb
docker exec mongodb-server mongodump `
  -u adminUser -p 12345 --authenticationDatabase admin `
  --db testdb `
  --out /data/db/backup

2025-10-28T14:07:36.362+0000    writing testdb.earthquakes to /data/db/backup/testdb/earthquakes.bson
2025-10-28T14:07:36.363+0000    writing testdb.users to /data/db/backup/testdb/users.bson
2025-10-28T14:07:36.373+0000    done dumping testdb.earthquakes (6 documents)
2025-10-28T14:07:36.376+0000    done dumping testdb.users (1 document)

docker run -it --rm --network mongo-network mongo:latest `
  mongosh --host mongodb-server -u adminUser -p 12345 --authenticationDatabase admin

use testdb

# Полностью удалим testdb
db.dropDatabase()

show dbs

admin   132.00 KiB
config  108.00 KiB
local    72.00 KiB

exit

# Восстановление из бекапа
docker exec mongodb-server mongorestore `
  -u adminUser -p 12345 --authenticationDatabase admin `
  --drop `
  /data/db/backup

2025-10-28T14:10:34.640+0000    7 document(s) restored successfully. 0 document(s) failed to restore.

docker run -it --rm --network mongo-network mongo:latest `
  mongosh --host mongodb-server -u dataUser -p 54321 --authenticationDatabase admin

# Всё успешно восстановилось
testdb> db.earthquakes.countDocuments()
6

# Изменим earthquakes.json оставим 2 записи: 1 проходит валидатор, 2 нет
# Сделаем импорт данных
docker exec mongodb-server mongoimport `
  -u adminUser  -p 12345 --authenticationDatabase admin `
  --db testdb `
  --collection earthquakes `
  --file /data/db/earthquakes.json `
  --jsonArray

# Валидация работает
2025-10-28T14:14:32.695+0000    connected to: mongodb://localhost/
2025-10-28T14:14:32.700+0000    continuing through error: Document failed validation
2025-10-28T14:14:32.701+0000    1 document(s) imported successfully. 1 document(s) failed to import.
~~~