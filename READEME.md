### Why would you consider a Scripting Language as JavaScript as your Backend Platform?

It can be a benefit to use the same langue on both frontend and backend, both in term experince and cost. 


### Explain Pros & Cons in using Node.js + Express to implement your Backend compared to a strategy using, for example, Java/JAX-RS/Tomcat

It is much more ligt weight to use express which is a server in iteself, then to use a tomcat server which can be a huge overkill for what is needed. You may also want to take into consideration that javascript is a single threaded langue, which can be a pro or a con depending on problem you want to solve.

### Node.js uses a Single Threaded Non-blocking strategy to handle asynchronous task. Explain strategies to implement a Node.js based server architecture that still could take advantage of a multi-core Server.

You deploy more instances of the backend, also called horizontal scaling. Horizontal scaling is about duplicating your application instance to manage a larger number of incoming connections. This action can be performed on a single multi-core machine or across different machines.

### Explain briefly how to deploy a Node/Express application including how to solve the following deployment problems

You download node on a server and your code base aswell(you could use git). then you could simply run the the program in the background. The issue with this is if the server goes down and need to restart, or the application gets a error. Then it wont start the app anew. There are alot of ways to solve this problem. One om them is to use a node module called pm2

if you need more then one app running on the same server you can use a reverse proxy like nginx

### Explain the difference between “Debug outputs” and application logging. What’s wrong with console.log(..) statements in our backend-code.

We only want debug messages to display in production

```javascript 
const app = express()
const winston = require('winston')
const consoleTransport = new winston.transports.Console()
const myWinstonOptions = {
    transports: [consoleTransport]
}
const logger = new winston.createLogger(myWinstonOptions)

function logRequest(req, res, next) {
    logger.info(req.url)
    next()
}
app.use(logRequest)

function logError(err, req, res, next) {
    logger.error(err)
    next()
}
app.use(logError)
``` 


### Explain, using relevant examples, concepts related to testing a REST-API using Node/JavaScript + relevant packages 

```javascript 
require('mocha')
const expect = require("chai").expect
const makeData = require("../makeData")
const dbDisconnect = require("../dbConnect").disconnect
const dbConnect = require("../dbConnect").connect
const restStart = require("./../rest/mainRest").start
const restStop = require("./../rest/mainRest").stop
const fetch = require("node-fetch")
const userModel = require("./../models/user")

describe("Test user rest endpoints", function () {
    before(async function () {
        await dbConnect();
        await restStart();
    })

    after(async function () {
        await restStop();
        await dbDisconnect();
    })

    beforeEach(async function () {
        await makeData();
    })

    describe("GET: user/all", function () {
        it("Should return return 3 users", async function () {
            const url = "http://localhost:3001/user/all";
            const response = await fetch(url);
            const data = await response.json();
            expect(data.length).to.be.equal(3)
        })
    })
    describe("GET: user/:username", function () {
        it("Should return user with username Perlt11", async function () {
            const url = "http://localhost:3001/user/Perlt11";
            const response = await fetch(url);
            const data = await response.json();
            expect(data.firstName).to.be.equal("Nikolai")
            expect(data.lastName).to.be.equal("Perlt")
        })
    })
    describe("POST: /", function () {
        it("Should post a new user", async function () {
            const url = "http://localhost:3001/user"
            const user = {
                firstName: "TEST",
                lastName: "TEST",
                userName: "TEST",
                password: "TEST",
                email: "TEST@gmail.com",
                job: [{
                    type: "TEST",
                    company: "TEST",
                    companyUrl: "TEST.net"
                }]
            }

            await fetch(url, {
                method: "POST",
                body: JSON.stringify(user),
                headers: {
                    "Content-Type": "application/json",
                }
            })
            const userList = await userModel.find({});
            expect(userList.length).to.be.equal(4)
        })
    })

    describe("POST: /job", async function () {
        it("Should create job on user", async function () {
            const url = "http://localhost:3001/user/job";
            const userId = await userModel.findOne({}).then(user => user._id);
            const body = {
                job: {
                    type: "TEST",
                    company: "TEST",
                    companyUrl: "TESt.net"
                },
                id: userId
            }
            await fetch(url, {
                method: "POST",
                body: JSON.stringify(body),
                headers: {
                    "Content-Type": "application/json",
                }
            })
            const user = await userModel.findById(userId);
            expect(user.job.length).to.be.equal(2);
        })
    })
})
```

### Explain, using relevant examples, the Express concept; middleware.

Middleware functions are functions that have access to the request object (req), the response object (res), and the next middleware function in the application’s request-response cycle. The next middleware function is commonly denoted by a variable named next.

Middleware functions can perform the following tasks:

Execute any code.
Make changes to the request and the response objects.
End the request-response cycle.
Call the next middleware function in the stack.
If the current middleware function does not end the request-response cycle, it must call next() to pass control to the next middleware function. Otherwise, the request will be left hanging.

### Compare the express strategy toward (server side) templating with the one you used with Java on second semester.

SPA is fast, as most resources (HTML+CSS+Scripts) are only loaded once throughout the lifespan of application. Only data is transmitted back and forth.

It’s easier to make a mobile application because the developer can reuse the same backend code for web application and native mobile application.

SPA can cache any local storage effectively. An application sends only one request, store all data, then it can use this data and works even offline.

### Demonstrate a simple Server Side Rendering example using a technology of your own choice (pug, EJS, ..).

```html
<!DOCTYPE html>
<html>
  <head>
    <title><%= title %></title>
    <link rel='stylesheet' href='/stylesheets/style.css' />
  </head>
  <body>
    <h1><%= title %></h1>
    <p>Welcome to <%= title %></p>
  </body>
</html>
```


```javascript 
var express = require('express');
var router = express.Router();
const debug = require('debug')("debugging-logging:route-index")
/* GET home page. */
router.get('/', function(req, res, next) {
  debug("Got a GET request")
  res.render('index', { title: 'Express' });
});

module.exports = router;
```

### Explain, using relevant examples, your strategy for implementing a REST-API with Node/Express and show how you can "test" all the four CRUD operations programmatically using, for example, the Request package.

Look at mini project


### Explain, using relevant examples, about testing JavaScript code, relevant packages (Mocha etc.) and how to test asynchronous code. 

```javascript 
require('mocha')
const expect = require("chai").expect
const makeData = require("../makeData")
const disconnect = require("../dbConnect").disconnect
const connect = require("../dbConnect").connect

const userFacade = require("../facades/userFacade")


describe("userFacade", function () {
    before(async function(){
        await connect();
    })

    after(async function(){
        await disconnect()
    })

    beforeEach(async function () {
        await makeData();
    })


    describe("addUser()", function () {
        it("Should create a new user", async function () {
            let user = {
                firstName: "test",
                lastName: "test",
                userName: "test",
                password: "test",
                email: "test@gmail.com",
                job: [{
                    type: "test",
                    company: "test",
                    companyUrl: "test.net"
                }]
            }
            let beforeUserList = await userFacade.getAllUsers()
            await userFacade.addUser(user);
            let afterUserList = await userFacade.getAllUsers()
            expect(afterUserList.length).to.be.equal(beforeUserList.length + 1);
        })
    })
    describe("getAllUsers()", function () {
        it("Should return all users and be length 3", async function () {
            const userAmount = await userFacade.getAllUsers().then(data => data.length);
            expect(userAmount).to.be.equal(3)
        })
    })

    describe("findByUsername", function(){
        it("should find user with userName Perlt11", async function(){
            const user = await userFacade.findByUserName("Perlt11");
            expect(user.firstName).to.be.equal("Nikolai")
            expect(user.lastName).to.be.equal("Perlt")
        })
    })

    describe("addJobToUser", function(){
        it("Should add job to user",async function(){
            const job = {
                type: "Test",
                company: "Test",
                companyUrl: "Test"
            }
            const user = await userFacade.findByUserName("Perlt11");
            const newUser = await userFacade.addJobToUser(user._id, job);
            expect(newUser.job.length).to.be.equal(2);
        })
    })
})

```


### Explain, using relevant examples, different ways to mock out databases, HTTP-request etc.

```typescript

import calc from './../src/calc'
import mim from './../src/makeItModular1'
import { server, resetJokes } from './../src/rest/rest'
import fetch from 'node-fetch'
import swapi from './../src/rest/swapi'
import nock from 'nock'

import { expect } from 'chai'
import 'mocha'
import fs from 'fs'
import rimraf from 'rimraf'

describe("Test swapi", function () {

    before(function () {
        nock("https://swapi.co").get("/api/people/1")
            .reply(200, JSON.stringify({
                name: "Luke Skywalker",
                height: "172",
                mass: "77"
            }))

        nock("https://swapi.co").get("/api/planets/1")
            .reply(200, JSON.stringify({
                name: "Tatooine",
                rotation_period: "23",
                orbital_period: "304"
            }))

        nock("https://swapi.co").get("/api/starships/1")
            .reply(200, JSON.stringify({
                name: "Death Star",
                model: "DS-1 Orbital Battle Station",
                manufacturer: "Imperial Department of Military Research, Sienar Fleet Systems"
            }))
    })

    describe("getPerson()", function () {
        it("Should return Anakin Skywalker", async function () {
            const person = await swapi.getPerson(1)
            expect(person.name).to.be.equal("Luke Skywalker")
        })
    })
    describe("getStarship()", function () {
        it("Should return Death Star", async function () {
            const person = await swapi.getStarship(1)
            expect(person.name).to.be.equal("Death Star")
        })
    })
    describe("getPlanet()", function () {
        it("Should return Tatooine", async function () {
            const person = await swapi.getPlanet(1)
            expect(person.name).to.be.equal("Tatooine")
        })
    })
})

```

### Explain, preferably using an example, how you have deployed your node/Express applications, and which of the Express Production best practices you have followed.

I have deployed with pm2, and used git to send code base to my server


### Explain, generally, what is meant by a NoSQL database. 

its a very wide term as it covers every database that is not relational.

### Explain Pros & Cons in using a NoSQL database like MongoDB as your data store, compared to a traditional Relational SQL Database like MySQL.

It is very fast to read data as you often avoid have to join tables. But it can also be slower to update because of the database is denormalized.

### Explain reasons to add a layer like Mongoose, on top on of a schema-less database like MongoDB

Becuase we often want a wat of controlling the content and format of the database

### Demonstrate, using a REST-API you have designed, how to perform all CRUD operations on a MongoDB


```javascript 

const User = require("../models/user")


async function addUser(user) {
    return await User.create(user)
}

async function addJobToUser(userId, job) { 
    return await User.findByIdAndUpdate(userId, {
        $push: {
            job
        }
    }, {new: true})
}

async function getAllUsers() {
    return await User.find({})
}

async function findByUserName(userName) {
    return await User.findOne({
        userName
    })
}

module.exports = {
    addUser,
    addJobToUser,
    getAllUsers,
    findByUserName
}

```


### Explain the benefits of using Mongoose, and demonstrate, using your own code, an example involving all CRUD operations

Look at the two questions above

### Explain the “6 Rules of Thumb: Your Guide Through the Rainbow” as to how and when you would use normalization vs. denormalization.

## one-to-few

```javascript
db.person.findOne()
{
  name: 'Kate Monster',
  ssn: '123-456-7890',
  addresses : [
     { street: '123 Sesame St', city: 'Anytown', cc: 'USA' },
     { street: '123 Avenue Q', city: 'New York', cc: 'USA' }
  ]
}
```
This design has all of the advantages and disadvantages of embedding. The main advantage is that you don’t have to perform a separate query to get the embedded details; the main disadvantage is that you have no way of accessing the embedded details as stand-alone entities.


### One-to-Many

```javascript
db.products.findOne()
{
    name : 'left-handed smoke shifter',
    manufacturer : 'Acme Corp',
    catalog_number: 1234,
    parts : [     // array of references to Part documents
        ObjectID('AAAA'),    // reference to the #4 grommet above
        ObjectID('F17C'),    // reference to a different Part
        ObjectID('D2AA'),
        // etc
    ]
```
Each Product would have its own document, which would contain an array of ObjectID references to the Parts that make up that Product:

### one-to-squillions

```javascript
db.logmsg.findOne()
{
    time : ISODate("2014-03-28T09:42:41.382Z"),
    message : 'cpu is on fire!',
    ipaddr : '127.66.66.66',
    host: ObjectID('AAAB')
}
```

You can also denormalize the “one-to-squillions” example. This works in one of two ways: you can either put information about the “one” side (from the 'hosts’ document) into the “squillions” side (the log entries), or you can put summary information from the “squillions” side into the “one” side.


# One:
 favor embedding unless there is a compelling reason not to
# Two:
 needing to access an object on its own is a compelling reason not to embed it
# Three:
 Arrays should not grow without bound. If there are more than a couple of hundred documents on the “many” side, don’t embed them; if there are more than a few thousand documents on the “many” side, don’t use an array of ObjectID references. High-cardinality arrays are a compelling reason not to embed.
# Four:
 Don’t be afraid of application-level joins: if you index correctly and use the projection specifier (as shown in part 2) then application-level joins are barely more expensive than server-side joins in a relational database.
# Five:
 Consider the write/read ratio when denormalizing. A field that will mostly be read and only seldom updated is a good candidate for denormalization: if you denormalize a field that is updated frequently then the extra work of finding and updating all the instances is likely to overwhelm the savings that you get from denormalizing.
# Six: 
As always with MongoDB, how you model your data depends – entirely – on your particular application’s data access patterns. You want to structure your data to match the ways that your application queries and updates it.

### Demonstrate, using your own code-samples, decisions you have made regarding → normalization vs denormalization 

```javascript 
const mongoose = require("mongoose");
const Schema = mongoose.Schema;

let JobSchema = new Schema({
    type: String,
    company: String,
    companyUrl: String
});

let UserSchema = new Schema({
    firstName: String,
    lastName: String,
    userName: {
        type: String,
        unique: true,
        required: true
    },
    password: {
        type: String,
        required: true
    },
    email: {
        type: String,
        required: true,
        unique: true
    },
    job: [JobSchema],
    created: {
        type: Date,
        default: Date.now
    },
    lastUpdated: Date
});
```
a user have a one-to-few job. therefore we have embedded the job data into user


### Explain, using a relevant example, a full JavaScript backend including relevant test cases to test the REST-API (not on the production database)

Look in miniproject

