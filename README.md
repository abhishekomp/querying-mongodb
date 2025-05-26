MongoDB local install on Mac (M4 Macbook Pro)
https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-os-x/
Installed using Homebrew

I performed mongo db installation on M4 Mac using homebrew and by taking some cues from this video:
https://www.youtube.com/watch?v=QEBuMROQuNw

Start mongodb that was installed using homebrew
brew services start mongodb/brew/mongodb-community
MongoDB website says this command for starting the service:

Following worked for me on Mac to start and stop the local instance of MongoDB (2025-05-23)
brew services start mongodb-community@8.0
brew services stop mongodb-community@8.0
To re-start the mongo db server:
brew services restart mongodb-community

MongoDB does not have any schema and does not encourages relationship between collections.
***************************************************************
My local installation on M4 Mac:
mongosh showed me this:
mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.5.1

MongoShell
Once the database has started, type the command mongosh which should open the mongo shell.
You can also connect to a particular Db Server by typing following command in a terminal window (worked for me)
mongosh --host localhost --port 27017
OR
mongosh “localhost:27017”
Show the current databases using the following command from mongosh command line: (https://www.youtube.com/watch?v=8gUQL2zlpvI)
To see the database to which we are connected to use the command db
show dbs
use testDb	(to select or work with a particular Database)
show collections
cls
cls is used to clear the screen in mongo shell

Another video: https://www.youtube.com/watch?v=6mmAvTwkVbA
This guy uses the download route but he also shows how to create a document using the command line.

To change the prompt when connected to MongoDB using mongo shell, we need to edit a file named .mongorc.js

To load your database with data from json file run this command:
mongoimport --file /Users/kjss920/Downloads/querying-data-mongodb/02/demos/sampledb/aircraft.json --collection aircraft --db flightmgmt --drop
mongoimport --file /Users/kjss920/Downloads/querying-data-mongodb/02/demos/sampledb/crew.json --collection crew --db flightmgmt --drop
mongoimport --file /Users/kjss920/Downloads/querying-data-mongodb/02/demos/sampledb/flights.json --collection flights --db flightmgmt --drop

--drop is optional to drop the existing collection

Imported a modified version(pushed to this repo as file name crew.json) of crew json in local mongodb using mongoimport command that worked fine for me on Mac
Failed: cannot decode array into a primitive.D
Changed the command as follows by including --jsonArray and then the modified json got imported fine:
mongoimport --file /Users/kjss920/Downloads/querying-data-mongodb/02/demos/sampledb/crew_modified.json --jsonArray --collection crew --db flightmgmt --drop

Note: Run this command from the terminal (not from mongosh)
Read this: The error SyntaxError: Missing semicolon typically indicates that you're trying to run the mongoimport command within the MongoDB shell (mongo), where it's not supported. The mongoimport command should be executed in your terminal or command prompt, not within the MongoDB shell. It's a separate tool used for importing data directly into your MongoDB instance.

Find
db.flights.countDocuments()
Note: countDocuments() is applied to a collection. I tried applying to the find command and got error.
db.aircraft.find()
Note by default mongoshell will display the documents in batches of 20 documents.
If you need to see the next set of documents then you need to type “it”

Format the output in mongoshell:
db.aircraft.find().pretty()

Syntax of find command
find(query, projection): cursor
Both are optional

Projection example:
db.flights.find({},{duration:1,"departure.city":1,"destination.city":1}).pretty()
In the above command we are saying to return all the documents (by specifying query as {}) from the flights collection and display the fields duration, departure.city and destination.city

Sorting example:
We can sort the output of the find command by invoking the sort method.
The sort method takes a sort object.
db.flights.find({},{duration:1,"departure.city":1,"destination.city":1}).pretty().sort({duration:-1})
-1 for descending order
1 for ascending order

Limit the number of documents from find command
db.aircraft.find().limit(2)

Query Filters and Query Operators

Anatomy of the find command:

find(query, projection): cursor
Both query and projection are optional for the find method.
The output of the find command is a cursor.
If you provide projection object, then you must also provide the query object even if the query object is just {}

findOne(query, projection): document
findOne() method Returns an actual document.
If no document matches the query, it returns null.
Note: pretty(), count() are not available for findOne() method

db.aircraft.find()
db.aircraft.find({})
The above 2 commands are actually same.
If you want to apply the projection then you will need to provide at least an empty query object(you cannot omit the query object in this case):
db.aircraft.find({}, {_id: 0}): This command has an empty query object.

Comparison query filters
The syntax of the query object in the find command is as follows:
{ field: { $operator: value } }
find( {field: {$gte: value}}, projection)
Example: 
db.aircraft.find({model: {$eq: "Boeing 737-900"}})
The shorthand for the above is:
db.aircraft.find({model: "Boeing 737-900”})

8 operators are: (Operator always start with a dollar sign)
$eq
$ne - not equal to
$in
$nin
$lt
$lte
$gt
$gte

Equality operator (can also use the shorthand version by not writing $eq)
find( {field : value}, projection)
db.aircraft.find({model: "Boeing 737-900”})
db.aircraft.find({model: {$eq: "Boeing 737-900"}})
The above 2 are same and are using the equality operator. The 1st one is short-hand version of equality operator usage.

db.aircraft.find({range: 5600})
db.aircraft.find({range: {$gt: 10000}})

Filter based on a boolean field, a date field, an id field
db.flights.find({delayed: true}, {delayed:1, duration: 1})
db.aircraft.find({_id: ObjectId("6830dc2bf6498f051a1503bf")})


Equality operator can also be used to filter array object. For example:
db.flights.find({crew: ["Attendant"]})
Note comparing array works like this. The above command will try to find those documents that have an array object named crew with only 1 value = Attendant. So if there is a document with an array named crew with more than 1 values then this document will not be matched by the above find command.

db.flights.find({"crew.position": "Attendant"}, {departureDate: 1, crew: 1})

Using not equal (ne)
db.aircraft.find({model: {$ne: "Boeing 737-900"}})
db.aircraft.find({capacity: {$gte: 200}})
db.aircraft.find({capacity: {$lt: 200}})

In/Non IN

find({field: {$in: [v1,v2]}}, projection)
Opposite is $nin
Example: Find all the aircrafts from the aircraft collection whose type is either International or Internal
db.flights.find({type: {$in: ["International", "Internal"]}}, {type: 1, duration: 1, _id: 0})
db.flights.find({type: {$nin: ["International", "Internal"]}}, {type: 1, duration: 1, _id: 0})

Note: Things become a bit tricky when you use $in operator with a field that is an array. Watch and learn.

$in can also be used with regular expression

Section 4: Creating Complex Queries

$and, $or

Find all aircrafts that meet all the following conditions:
- Capacity is 124
- Range is greater than 6000

find({ $and: [ {capacity: 124}, {range: {$gt: 6000}} ] }, projection)
find({ $and: [ {capacity: 124}, {range: {$gt: 6000}} ] }, {model: 1, range: 1, capacity: 1})

flightmgmt> db.aircraft.find({ $and: [ {capacity: 124}, {range: {$gt: 6000}} ] }, {model: 1, range: 1, capacity: 1})
[
  {
    _id: ObjectId('6830dc2bf6498f051a1503c4'),
    model: 'Airbus A319',
    range: 6900,
    capacity: 124

  }
]

Querying nested document 
(remember to use quotation marks around the field name in this case)

Null fields, Missing fields
db.crew.find( address: null ).pretty()
Query will return all documents where address field is null and also those documents where address field does not exist. So pay attention about this behaviour.
So if you want to find all documents where the address has “null” value then use this:
db.crew.find({address: {$type: “null”}})

$exists
db.crew.find( address: {$exists: false } ).pretty()
This query will return all the documents where the address field does not exist.
db.crew.find( address: {$exists: true } ).pretty()
The above query will return all the documents where the address field exists. (It may have “null” value as well).
Okay, what if we want to return all the documents where the address field exists and have a non-null value.
db.crew.find( address: {$exists: true, $type: “string” } ).pretty()
Here we are saying that the address field must have a value of type string.

$type
Used to search for documents where the value of a field is of particular type (actually BSON type)
Find all the documents which has an address field of type Object
db.crew.find({address: {$type: “object”}})
db.crew.find({address: {$type: 3}})
Above 2 have the same meaning.
Find all the documents where a field has a value null. In the below query string we are finding those documents whose role field has the value null.
db.crew.find({role: {$type: "null"}}, {name:1, role:1, _id:0})

Working with Arrays

We may have following 2 scenarios when querying array
1. When you want to query for all documents where an array must have the “one of the values” as specified in the query.
2. When you want to query for documents where an array’s all values matches the one passed in the query.

db.crew.find( {skills: “technical”} )
db.crew.find( {skills: [“technical”]} )
Notice the difference: In the 1st example the query will fetch all those documents where the skills array has an element with value “technical”. However, if you use the 2nd expression then it will mean to fetch all documents where the array is exactly matching with the provided array, in this case the provided array is [“technical”]

Array Query Operators

$all
$size
$elemMatch

db.crew.find( {skills: {$all: [“technical”, “sales”]}} )
When using $all, the sequence of elements do not matter while searching, but the sequence in which you provide the array elements in the query DO MATTER when using equality comparion.
For example, the following is an equality check
db.crew.find( {skills: [“technical”, “sales”]} )

$elemMatch (IMP)
If the array elements are objects then querying requires using elemMatch in certain cases
Watch the video

Array Projection Operators

Printing just first element from the array
$slice
************************************************
Questions related to the Section 2: Writing your First Mongo Query

1. Display all the documents from the aircraft collection.
2. Display all the documents from the aircraft collection but display only the fields model, range and capacity.
3. Display all the documents from the aircraft collection and just display the model.
4. Display the first 2 aircraft documents
5. Display the aircraft documents sorted by capacity value in descending order.
6. Display the count of aircrafts in the collection.
7. Display the duration, departure city, destination city from the flights collection.


Answers:
1. db.aircraft.find() OR db.aircraft.find({})
2. db.aircraft.find({}, {model:1,range:1,capacity:1})
3. db.aircraft.find({},{model:1, "_id":0}) OR db.aircraft.find({},{model:1, _id:0})
4. db.aircraft.find().limit(2)
5. db.aircraft.find({}, {model: 1, _id: 0, capacity: 1}).sort({capacity: -1}) (for ascending order use 1 and for descending order use -1)
6. db.aircraft.countDocuments()
7. db.flights.find({}, {duration: 1, "destination.city": 1, "departure.city": 1, _id:0})
************************************************
Questions related to the Section 3: Understanding Query Filters and Query Operators

1. Find flight by id 6830dbfa015dbbad409fd9ac
2. Find flights that are delayed. Display only delayed and departure city for those flights.
3. Number of non-internal flights
4. Flights that depart after 2020-02-21
5. Flights shorter than 3 hours and display them in ascending order.

Answers:
1. db.flights.find({_id: ObjectId("6830dbfa015dbbad409fd9ac")})
2. db.flights.find({delayed: true}, {delayed: 1, "departure.city": 1, _id: 0})
3. db.flights.find({type: {$ne: "Internal"}},{type: 1, duration: 1}) Actually: db.flights.countDocuments({type: {$ne: "Internal"}})
4. db.flights.find({departureDate: {$gt: ISODate("2020-02-21")}},{departureDate:1,duration:1})
	Note: We are also using projection here.
	For finding flights with departure date less than a given date:
	db.flights.find({departureDate: {$lt: ISODate("2020-02-21")}},{departureDate:1,duration:1})
5. db.flights.find({duration: {$lt: 180}}, {duration:1, type:1}).sort({duration:1})
************************************************
Crew collection (modified for my purpose) 
is a custom collection I created that has following features:
1. An array of string and in 1 document this array is empty. In other documents the array may have 1 or 2 string entries.
2. role field may be null in some of the documents.
3. flyingHours field is present in only those documents whose role is either Pilot or Chief Pilot.

Questions related to Section 4: Creating Complex Queries

1. Find all flights whose departure city is Paris and distanceKm is greater than 1500
2. Find all aircraft whose range is between 3000 and 6000
3.  Find all the aircraft where the range is greater than 6000 OR the capacity is greater than 200
4. Find all the crew members having address city as Gothenburg (There should be 2 such crew)
5. Find all the crew members whose role is null (We need only those documents where the role is null, not those documents where the role is not at all present)
6. Find all the crew members where the role field doesn’t exists.
7. Find all the crew members whose role is not null
8. Find all the crew members who don’t have flyingHours field.
9. Find all the crew members who have flyingHours field.
10.  Find all the crew members (all such documents) where the address field is of type object.
11.  Find all the crew members (all such documents) where the crew has exactly 1 skill.
12.  Find all the crew members (all such documents) where the crew has exactly 0 skill.
13.  Find all the flights that depart from Paris
14.  Find all the fligths that meet all the conditions:
-   duration less than 2 hours
-   internal flights
15. Find all the flights that meet any condition:
-   depart from Germany
-   land in Germany
16.  Find all the flights where the aircraft code exists as a field
17.  Find all the flights where the aircraft code exists and is of type string



Answers:
1. db.flights.find({$and: [{"departure.city": {$eq: "Paris"}}, {distanceKm: {$gt: 1500}}]}, {type:1,"departure.city":1,distanceKm:1})
2. db.aircraft.find({$and: [{range: {$gt: 3000}}, {range: {$lt: 6000}}]}, {range:1, capacity:1})
	A shortcut for this kind of query when you are querying the same field twice is as follows:
	db.aircraft.find({range: {$gt:3000,$lt:6000}},{range:1,model:1})
3. db.aircraft.find({$or: [{capacity:{$gt:200}},{range: {$gt: 6000}}]}, {capacity:1, range: 1})
4. db.crew.find({"address.city": "Gothenburg"},{name:1,address:1})
5. db.crew.find({role: null},{role:1,name:1}). Note this query will even return those documents where the role field doesn’t exist. If you want only those documents where role exists and have a null value then use the following query db.crew.find({$and: [{role:null},{role: {$exists: true}}]},{role:1,name:1}) Another way to do this is by using $type operator to match the type of the value for the role field. The query looks like this: db.crew.find({role: {$type: "null"}}, {name:1, role:1, _id:0})
6. db.crew.find({role: {$exists: false}},{role:1,name:1})
7. db.crew.find({role: {$ne: null}},{role:1,name:1})
8. db.crew.find({flyingHours: {$exists: false}},{name:1,role:1})
9. db.crew.find({flyingHours: {$exists: true}}, {name:1, role: 1, flyingHours:1})
10.  db.crew.find({address: {$type: "object"}},{name:1,address:1})
11.  db.crew.find({skills: {$size: 1}}, {name:1,skills:1})
12.  db.crew.find({skills: {$size: 1}}, {name:1,skills:0})
