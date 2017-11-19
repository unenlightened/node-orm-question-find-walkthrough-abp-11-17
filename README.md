# Node Question ORM Find Walkthrough

## Objectives

1. Define a `Question.Find(id)` static function for `Question`
2. Write an `SELECT` SQL statement with a `WHERE` clause to find a question by primary key.
3. Use SQL replacement for sanitation in your SQL statement.
4. Return a `Promise` for the `Question.Find(id)` function.
6. Execute a SQL Statement using the `sqlite3` NPM package with `db.get()`
7. Reify a `Question` instance based on the result from the database.

## Instructions

In `models/Question.js` there is an ORM `Question` class that implements migration method for its SCHEMA `.CreateTable()` and a `constructor` that defines a `content` property for instances, and `insert()` which will persist an instance of a Question to the database.

The goal is to build a static class function, `Find(id)` that can return an instance of a question by searching the database for a question with a match primary key `id`.

In order to function properly, the database execution must be wrapped in a `Promise` that resolves. 

In the callback of `db.get()`, take the resulting row and use the data from the result to create a new instance of a Question that mirrors the row by assigning the corresponding properties of the result row to the new question instance.

Resolve the promise by returning the question instance.

**SPOILER: Below you will find Walkthrough Instructions for solving the lab**

## Walkthrough Instructions

**These instructions are progressive and if you follow them you will solve the lab. Only the final code block is the solution, all other code blocks are building up to it.**

Our `Question` class begins below.

**File: [models/Question.js](models/Question.js)**
```js
const db = require("../config/db")

class Question{
  static CreateTable() {
    return new Promise(function(resolve){
      const sql = `CREATE TABLE questions (
        id INTEGER PRIMARY KEY,
        content TEXT
      )`
      
      db.run(sql, function(){
        resolve("questions table created")
      })      
    })
  }

  constructor(content){
    this.content = content
  }

  insert(){
    const self = this 
    const sql = `INSERT INTO questions (content) VALUES (?)`
    return new Promise(function(resolve){
      db.run(sql, [self.content], function(err, result){
        self.id = this.lastID
        resolve(self)      
      })
    })
  }

}


module.exports = Question;
```

The first thing to do is stub out our `Find(id)` class function.

**File: [models/Question.js](models/Question.js)**
```js
const db = require("../config/db")

class Question{
  static CreateTable() {
    return new Promise(function(resolve){
      const sql = `CREATE TABLE questions (
        id INTEGER PRIMARY KEY,
        content TEXT
      )`
      
      db.run(sql, function(){
        resolve("questions table created")
      })      
    })
  }

  constructor(content){
    this.content = content
  }

  insert(){
    const self = this 
    const sql = `INSERT INTO questions (content) VALUES (?)`
    return new Promise(function(resolve){
      db.run(sql, [self.content], function(err, result){
        self.id = this.lastID
        resolve(self)      
      })
    })
  }

  static Find(id){

  }
}

module.exports = Question;
```

Running the tests in this state produces:

```
  Question
    .CreateTable()
      ✓ is a static class function
      ✓ returns a promise
      ✓ creates a new table in the database named 'questions'
      ✓ adds 'id' and 'content' columns to the 'questions' table (59ms)
    insert()
      ✓ is a function
      ✓ returns a promise
      ✓ inserts the row into the database
      ✓ sets the id of the instance based on the primary key
      ✓ returns the instance as the resolution of the promise
    .Find()
      ✓ is a static class function
      1) returns a promise


  10 passing (292ms)
  1 failing

  1) Question
       .Find()
         returns a promise:
     AssertionError: expected undefined to be an instance of Promise
      at Context.<anonymous> (test/models/QuestionTest.js:134:36)
      at <anonymous>
```

Focusing only on the first error, we know that ultimately `Find(1)` should be returning a promise. Let's implement that as naively as possible.

**File: [models/Question.js](models/Question.js)**
```js
const db = require("../config/db")

class Question{
  static CreateTable() {
    return new Promise(function(resolve){
      const sql = `CREATE TABLE questions (
        id INTEGER PRIMARY KEY,
        content TEXT
      )`
      
      db.run(sql, function(){
        resolve("questions table created")
      })      
    })
  }

  constructor(content){
    this.content = content
  }

  insert(){
    const self = this 
    const sql = `INSERT INTO questions (content) VALUES (?)`
    return new Promise(function(resolve){
      db.run(sql, [self.content], function(err, result){
        self.id = this.lastID
        resolve(self)      
      })
    })
  }

  static Find(id){
    return new Promise(function(resolve){
      resolve("This should be a found question")
    })
  }
}

module.exports = Question;
```

Running the tests again in the state above shows:

```
  Question
    .CreateTable()
      ✓ is a static class function
      ✓ returns a promise
      ✓ creates a new table in the database named 'questions'
      ✓ adds 'id' and 'content' columns to the 'questions' table
    insert()
      ✓ is a function
      ✓ returns a promise
      ✓ inserts the row into the database
      ✓ sets the id of the instance based on the primary key
      ✓ returns the instance as the resolution of the promise
    .Find()
      ✓ is a static class function
      ✓ returns a promise
      1) finds a Question in the database by its ID and returns a Question object


  11 passing (161ms)
  1 failing

  1) Question
       .Find()
         finds a Question in the database by its ID and returns a Question object:
     AssertionError: expected 'This should be a found question' to be an instance of Question
      at Context.<anonymous> (test/models/QuestionTest.js:143:38)
      at <anonymous>
```

The test expected an instance of a Question to be returned, let's continue with our naive implementation and return an empty new Question instance.

**File: [models/Question.js](models/Question.js)**
```js
const db = require("../config/db")

class Question{
  static CreateTable() {
    return new Promise(function(resolve){
      const sql = `CREATE TABLE questions (
        id INTEGER PRIMARY KEY,
        content TEXT
      )`
      
      db.run(sql, function(){
        resolve("questions table created")
      })      
    })
  }

  constructor(content){
    this.content = content
  }

  insert(){
    const self = this 
    const sql = `INSERT INTO questions (content) VALUES (?)`
    return new Promise(function(resolve){
      db.run(sql, [self.content], function(err, result){
        self.id = this.lastID
        resolve(self)      
      })
    })
  }

  static Find(id){
    return new Promise(function(resolve){
      const question = new Question()
      resolve(question)
    })
  }
}

module.exports = Question;
```

Another test run gives us:

```
  Question
    .CreateTable()
      ✓ is a static class function
      ✓ returns a promise
      ✓ creates a new table in the database named 'questions'
      ✓ adds 'id' and 'content' columns to the 'questions' table
    insert()
      ✓ is a function (40ms)
      ✓ returns a promise
      ✓ inserts the row into the database
      ✓ sets the id of the instance based on the primary key
      ✓ returns the instance as the resolution of the promise
    .Find()
      ✓ is a static class function
      ✓ returns a promise
      ✓ finds a Question in the database by its ID and returns a Question object
      1) returns a Question object with the correct properties from the DB


  12 passing (363ms)
  1 failing

  1) Question
       .Find()
         returns a Question object with the correct properties from the DB:
     AssertionError: expected undefined to deeply equal 1
```     

The stubbed out question instance we're returning lacks the properties of the question from the database. 

At this point the structure of the function is good but we have to actually interact with the database now and use `SELECT` to find the corresponding row in order to fill the question of the instance with the correct data.

Let's write the SQL first:

```sql
SELECT * FROM questions WHERE id = ?
```

The `?` represents the `id` of the question we are looking for and we'll use sql sanitation to replace that value with the `id` argument passed to `Question.Find(id)`.

With that we can implement the first part of the DB execution call with the `db.get()` function.



**File: [models/Question.js](models/Question.js)**
```js
const db = require("../config/db")

class Question{
  static CreateTable() {
    return new Promise(function(resolve){
      const sql = `CREATE TABLE questions (
        id INTEGER PRIMARY KEY,
        content TEXT
      )`
      
      db.run(sql, function(){
        resolve("questions table created")
      })      
    })
  }

  constructor(content){
    this.content = content
  }

  insert(){
    const self = this 
    const sql = `INSERT INTO questions (content) VALUES (?)`
    return new Promise(function(resolve){
      db.run(sql, [self.content], function(err, result){
        self.id = this.lastID
        resolve(self)      
      })
    })
  }

  static Find(id){
    const sql = "SELECT * FROM questions WHERE id = ?"
    return new Promise(function(resolve){
      db.get(sql, [id], function(err, result){
        const question = new Question(result.content)
        question.id = result.id
        resolve(question)
      })
    })
  }
}

module.exports = Question;
```

The snippet here is the full implementation of the database interaction:

```js
  db.get(sql, [id], function(err, result){
    const question = new Question()
    question.result = result.content
    question.id = result.id
    resolve(question)
  })
```      

`db.get()` accepts 3 arguments, the first the `sql` statement, the second is the replacement for the `?` in our SQL statement, the third is the callback with the result row.

Within that callback we cast the properties from `result` to the corresponding properties of the `question` instance and `resolve` the promise to return the `question`.

A final test run:

```
  Question
    .CreateTable()
      ✓ is a static class function
      ✓ returns a promise
      ✓ creates a new table in the database named 'questions'
      ✓ adds 'id' and 'content' columns to the 'questions' table
    insert()
      ✓ is a function
      ✓ returns a promise
      ✓ inserts the row into the database
      ✓ sets the id of the instance based on the primary key
      ✓ returns the instance as the resolution of the promise
    .Find()
      ✓ is a static class function
      ✓ returns a promise
      ✓ finds a Question in the database by its ID and returns a Question object
      ✓ returns a Question object with the correct properties from the DB (53ms)


  13 passing (282ms)
```

The ORM is becoming mroe and more complete.