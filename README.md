## SEIR 0508

# Express Router


In our first Express apps, we used the `app.get` method to define routes and although it worked, the better practice is to:

- Use Express `router` objects to define routes for a particular purpose or dedicated to a certain data resource such as `tasks`, `cats`, `users`.
- Create each `router` in its own module from which it is exported.
- Inside of **server.js** `require` and mount the `router` object in the request pipeline.

> Note:  A **data resource** is a "type" of data/information that applications create, read, update and/or delete (CRUD).

### Best Practice Routing Set Up by Express Generator

As an example of using this better approach to routing, let's look at how `express-generator` sets up routing...

First, there's a `routes` folder containing two router modules:

- **index.js**: Great for defining general purpose routes, e.g., the root route.
- **users.js**: An example of a router dedicated to a _data resource_, in this case, _users_. 

Note how routes are defined on those two `router` objects using `router.get()` method call just like we did previously with `app.get()`

Each `router` object has one route defined - compare those two routes, notice the **HTTP methods** and the **paths**?  They're the same - isn't that a problem?  Nope, they're not actually the same because of the way the routers are mounted in **server.js**...

### Router Objects in the Scaffolded App

The two route modules are required on lines 7 & 8 of `server.js`.

Then those routers are mounted in the middleware pipeline with the `app.use` method on lines 22 & 23:

```js
app.use('/', indexRouter);
app.use('/users', usersRouter);
```

> **IMPORTANT KEY POINT:** The path specified in `app.use` is a **"starts with path"**. It is **prepended** to the paths specified in the router object forming the **actual path**.

### Determining the Actual Path of a Route Defined in a Router Object

Let's say you have a `router` object that defines a route like this:

```js
// routes/todos.js

var express = require('express');
var router = express.Router();

router.get('/', function(req, res) {...
```
	
and is mounted like this:
	
```js
const todosRouter = require('./routes/todos');

// All routes defined in todosRouter will start with /todos
app.use('/todos', todosRouter);
```

<details>
<summary>
❓ What is the actual path of the route?
</summary>
<hr>

The starts with path is `/todos` and the path of the defined route is just `/` which doesn't change the actual path, thus the actual path is `/todos`

<hr>
</details>

Another example, let's say you have a `router` object that defines a route like this:

```js
// routes/calendar.js

var express = require('express');
var router = express.Router();

router.get('/today', function(req, res) {...
```
	
and is mounted like this:
	
```js
const calendarRouter = require('./routes/calendar');

app.use('/calendar', calendarRouter);
```

<details>
<summary>
❓ What is the actual path of the above route?
</summary>
<hr>

The starts with path is `/calendar` and the path of the defined route is `/today` making the actual path `/calendar/today`

<hr>
</details>



- We now want to define the route for the To-Dos **index** functionality (display all To-Dos).  However, we are not going to write an anonymous inline function for the route handler.  Instead, we are going to follow a best practice of putting the function in a controller module that can export any number of controller actions (functions).

- Here's the route that uses a controller action that we'll code in a bit:

	```js
	// routes/todos.js

	var express = require('express');
	var router = express.Router();

	// Require the controller that exports To-Do CRUD functions
	var todosCtrl = require('../controllers/todos');

	// All actual paths begin with "/todos"

	// GET /todos
	router.get('/', todosCtrl.index);
	```
	> Is that route definition tidy or what?!?!
