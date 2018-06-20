## Full Stack Express/Knex App, simplified

In this guide substitute `projectname`, `dbname`, `tablename`, and `colnameN` for your own values.

---

### Express Setup

1. New repo on GitHub, choose .gitignore: Node, a license, and clone it to your machine
1. sh: `cd projectname && atom .`
1. sh: `express --git --view ejs`
1. sh: `npm i && nodemon`
1. All good if `http://localhost:3000` loads
1. Ensure .gitignore has:
```
node_modules/
.DS_Store
.env
yarn.lock
package-lock.json
```
1. Create a `.eslintrc` in project root if you wish. If so, also run sh: `npm i --save-dev eslint`
1. git add, commit, push
---

### Database Setup

1. Ensure you have knex and postgreSQL installed globally on your machine. Test by typing `knex` and `psql` respectively, into your shell.
1. sh: `createdb dbname-dev` to create the dev database in psql
1. sh: `npm i --save knex pg` to install knex + postgres modules for your project
1. Create a `knex.js` in your project root with the following:
```javascript
// Require knex + detect environment
const environment = process.env.NODE_ENV || 'development'
const knexConfig = require('./knexfile')[environment]
const knex = require('knex')(knexConfig)
module.exports = knex
```
1. Create a `knexfile.js` in your project root with the following:
```javascript
// Define DB connections for different environments
module.exports = {
  development: {
    client: 'pg',
    connection: 'postgres://localhost/dbname-dev'
  },
  test: {},
  production: {
    client: 'pg',
    connection: process.env.DATABASE_URL
  }
}
```
1. git add, commit, push
---

### Migrations (Define DB Schema)

1. sh: `knex migrate:make tablename` to make migration file for a db table
1. Find the migration file it just created and make it look like:
```javascript
exports.up = function(knex, Promise) {
  return knex.schema.createTable('tablename', function(table) {
    // TABLE COLUMN DEFINITIONS HERE
    table.increments()
    table.string('colname1', 255).notNullable().defaultTo('')
    table.string('colname2', 255).notNullable().defaultTo('')
    table.string('colname3', 255).notNullable().defaultTo('')
    table.timestamps(true, true)
    // OR
    // table.dateTime('created_at').notNullable().defaultTo(knex.raw('now()'))
    // table.dateTime('updated_at').notNullable().defaultTo(knex.raw('now()'))
  })
}
exports.down = function(knex, Promise) {
  return knex.schema.dropTableIfExists('tablename')
}
```
1. Repeat the last 2 steps for each table in your ERD
1. sh: `knex migrate:latest` to run all migrations, creating the tables in your psql db
1. If `knex migrate:latest` messed something up you can always `knex migrate:rollback`
1. sh: `psql dbname-dev` to verify it created your tables correctly
1. In psql: `\d tablename`
1. git add, commit, push
---

### Seeds (Insert DB Data)

1. sh: `knex seed:make 001_tablename`
1. Edit the seed file it created for your table to look similar to this:
```javascript
exports.seed = function(knex, Promise) {
  // Deletes ALL existing entries
  return knex('tablename').del()
    .then(function() {
      // Inserts seed entries
      return knex('tablename').insert([
        {id: 1, colname1: '', colname2: '', colname3: ''},
        {id: 2, colname1: '', colname2: '', colname3: ''},
        {id: 3, colname1: '', colname2: '', colname3: ''}
      ])
      .then(function() {
        // Moves id column (PK) auto-incrementer to correct value after inserts
        return knex.raw("SELECT setval('tablename_id_seq', (SELECT MAX(id) FROM tablename))")
      })
    })
}
```
1. Repeat the last 2 steps for each table in your ERD. Give each seed file a unique 3 digit prefix so you can control the order in which seeding occurs.
1. sh: `knex seed:run` to insert all seed data into your database
1. sh: `psql dbname-dev` to verify it seeded correctly
1. In psql: `SELECT * FROM tablename`
1. You can always change your seed files and rerun: `knex seed:run`
1. git add, commit, push
---

### Routes

1. In your project root under `routes/`, create a new file `tablename.js`
1. Setup your basic routes for that file:
```javascript
const express = require('express')
const router = express.Router()
const knex = require('../knex')
// READ ALL records for this table
router.get('/', (req, res, next) => {
  res.send('ALL RECORDS')
})
// READ ONE record for this table
router.get('/:id', (req, res, next) => {
  res.send('ONE RECORD')
})
// CREATE ONE record for this table
router.post('/', (req, res, next) => {
  res.send('CREATED RECORD')
})
// UPDATE ONE record for this table
router.put('/:id', (req, res, next) => {
  res.send('UPDATED RECORD')
})
// DELETE ONE record for this table
router.delete('/:id', (req, res, next) => {
  res.send('DELETED RECORD')
})
module.exports = router
```
1. In your `app.js` make to require the route file:
```javascript
var tablenameRouter = require('./routes/tablename')
```
and to use it:
```javascript
app.use('/tablename', tablenameRouter)
```
1. sh: `nodemon` and test our GET/POST/PUT/DELETE routes with HTTPie
1. If all is well git add, commit, push
1. Go back to our route file. Make the GET routes work with Knex:
```javascript
// READ ALL records for this table
router.get('/', (req, res, next) => {
  knex('tablename')
    .then((rows) => {
      res.json(rows)
    })
    .catch((err) => {
      next(err)
    })
})
// READ ONE record for this table
router.get('/:id', (req, res, next) => {
  knex('tablename')
    .where('id',req.params.id)
    .then((rows) => {
      res.json(rows)
    })
    .catch((err) => {
      next(err)
    })
})
```
1. Make the POST route work with Knex:
```javascript
// CREATE ONE record for this table
router.post('/', (req, res, next) => {
  knex('tablename')
    .insert({
      "colname1": req.body.colname1,
      "colname2": req.body.colname2,
      "colname3": req.body.colname3
    })
    .returning('*')
    .then((data) => {
      res.json(data[0])
    })
    .catch((err) => {
      next(err)
    })
})
```
1. Make the PUT route work with Knex:
```javascript
// UPDATE ONE record for this table
router.put('/:id', (req, res, next) => {
  knex('tablename')
  .where('id', req.params.id)
  .then((data) => {
    knex('tablename')
    .where('id', req.params.id)
    .limit(1)
    .update({
      "colname1": req.body.colname1,
      "colname2": req.body.colname2,
      "colname3": req.body.colname3
    })
    .returning('*')
    .then((data) => {
      res.json(data[0])
    })
  })
  .catch((err) => {
    next(err)
  })
})
```
1. Make the DELETE route work with Knex:
```javascript
// DELETE ONE record for this table
router.delete('/:id', function(req, res, next) {
  knex('tablename')
    .where('id', req.params.id)
    .first()
    .then((row) => {
      if(!row) return next()
      knex('tablename')
        .del()
        .where('id', req.params.id)
        .then(() => {
          res.send(`ID ${req.params.id} Deleted`)
        })
    })
    .catch((err) => {
      next(err)
    })
})
```
1. These routes do not have any real input validation or defensive coding going on. A real app will need this to prevent shenanigans from users.
1. Test these routes with HTTPie and your database tool `psql`
1. If all is well, git add, commit, push
---

### Frontend (SPA)

1. Only one view needed: `views/index.ejs`
1. Put frontend JS in `public/javascripts/scripts.js`
1. Put CSS in `public/stylesheets/styles.css`
1. Reference both of these from `views/index.ejs` with script/link tags
1. In `public/javascripts/scripts.js` add AJAX code to request JSON from our backend routes.
```javascript
$(document).ready( function () {
  $('#btn-go').click((event) => {
    $.ajax({
      url: '/tablename',
      type: 'GET',
      success: (data) => {
        console.log(data)
        // UPDATE DOM!
        $('doohickey').append(data)
      },
      error: function(jqXhr, textStatus, errorThrown) {
        console.log('OOPS:', errorThrown)
      }
    })
  })
})
```
1. We write frontend AJAX like this against our backend routes (API) in various ways and update our DOM accordingly.
---

### Frontend (Templates)

1. Create templates under `views/` to to render HTML, and change all `res.json()` / `res.send()` calls in routes to `res.render()` calls
1. Duplicate `views/index.ejs` and rename to `tablename.ejs`
1. Utilize template tags and `res.render()` to push data to your templates
---
