*MONGODB CONVERT "JSON" FORMAT TO ANOTHER FORMAT FOR STORING, CALLED "BSON" (it is binary version of json and can queried more efficiently)*

 - Default Storage Engine is "Wired Tiger"
 - Storage Engine loads a chunk of data into memory and manages to store the data in memory that is used frequently.
 - No Schema
 - No/Few Relations
 - MongoDB has a couple of hard limits - most importantly, a single document in a collection (including all embedded documents it might have) must be <= 16mb. 
 - Additionally, you may only have 100 levels of embedded documents.


**update vs updateMany**
```js
db.flightData.update({_id:ObjectId("5da7ab9862623af364fe2d43")},{delayed: false})
```
> (if $set is not used) : this command will replace the entire document with the second parameter, but the _id remains the same. But updateOne will not work without $set.

```js
{
	"_id" : ObjectId("5da99d383082c289904c82e8"),
	"name" : "Albert Twostone",
	"age" : 68,
	"hobbies" : [
		"sports",
		"cooking"
	]
}

db.passengers.findOne({hobbies: "sports"});    
/* (mongoDB automatically check that "sports" is present in hobbies array 
and then return that docement) */
```

## DATA TYPES IN MONGODB

| Data Type         	| E.g                   	|                                 	|
|-------------------	|-----------------------	|---------------------------------	|
| Text              	| Some text             	|                                 	|
| Boolean           	| true / false          	|                                 	|
| Number            	| 55                    	| NumberInt(int32),signed integer 	|
|                   	| 10000000000000        	| NumberLong(int64)               	|
|                   	| 12.39                 	| NumberDecimal()                 	|
| ObjectId          	| ObjectId("asdfa")     	|                                 	|
| ISODate           	| ISODate("2019-10-19") 	|                                 	|
| Timestamp         	| Timestamp(113477747)  	|                                 	|
| Embedded document 	| { "a": {....}}        	|                                 	|
| Array             	| { "b": [....]}        	|                                 	|


```js
typeof db.numbers.findOne({}).a // number
```

- NumberInt creates a int32 value => NumberInt(55)
- NumberLong creates a int64 value => NumberLong(7489729384792) 
- Number stored in mongodb can be atmost 64 bit long (it is the maximum limit)
-  If you just use a number (e.g. insertOne({a: 1}), this will get added as a normal double into the database. The reason for this is that the shell is based on JS which only knows float/ double values and doesn't differ between integers and floats.
- NumberDecimal creates a high-precision double value => NumberDecimal("12.99") => This can be helpful for cases where you need (many) exact decimal places for calculations.


## Joining with $lookup

```js
db.books.aggregate([{$lookup:{
    from: "authors",       //from which colection you want to relate document
    localField: "authors", //authors is an array of ObjectId
    foreignFiled: "_id",  //name of the field of "authors" collecion
    as: "creators",       //alias name
}}]);
```

## Schema Validation
```js

db.createCollection('posts', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['title', 'text', 'creator', 'comments'],
      properties: {
        title: {
          bsonType: 'string',
          description: 'must be a string and is required'
        },
        text: {
          bsonType: 'string',
          description: 'must be a string and is required'
        },
        creator: {
          bsonType: 'objectId',
          description: 'must be an objectid and is required'
        },
        comments: {
          bsonType: 'array',
          description: 'must be an array and is required',
          items: {
            bsonType: 'object',
            required: ['text', 'author'],
            properties: {
              text: {
                bsonType: 'string',
                description: 'must be a string and is required'
              },
              author: {
                bsonType: 'objectId',
                description: 'must be an objectid and is required'
              }
            }
          }
        }
      }
    }
  }
});

# Modifing collecion usind runCommand and collMod(collecion modifier)

db.runCommand({
  collMod: 'posts',
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['title', 'text', 'creator', 'comments'],
      properties: {
        title: {
          bsonType: 'string',
          description: 'must be a string and is required'
        },
        text: {
          bsonType: 'string',
          description: 'must be a string and is required'
        },
        creator: {
          bsonType: 'objectId',
          description: 'must be an objectid and is required'
        },
        comments: {
          bsonType: 'array',
          description: 'must be an array and is required',
          items: {
            bsonType: 'object',
            required: ['text', 'author'],
            properties: {
              text: {
                bsonType: 'string',
                description: 'must be a string and is required'
              },
              author: {
                bsonType: 'objectId',
                description: 'must be an objectid and is required'
              }
            }
          }
        }
      }
    }
  },
  validationAction: 'warn'
});
```

## mongod flags:

```html
sudo mongod --help
sudo mongod --port 27020
sudo mongod --dbpath "/var/lib/mongodb"
sudo mongod --logpath "/home/ankur/logs/log.log"
sudo mongod --fork    (to run mongodb as a background service)   ===>   use db.shutdownServer() to stop the server running in
                                                                        background
sudo mongod --config (-f) "path to config file"
help command inside mongo shell                                                                    
```



## IMPORTING DATA TO MONGODB FROM A JSON FILE

```html
Query: mongoimport <path to json file> -d <DATABASE NAME> -c <COLLECTION NAME> --jsonArray --drop

        -d => represent database
        -c => collecion
        --jsonArray => tell mongodb that the data is a jsonArray
        --drop => it will drop the collection if it already exist 
```                                                     


