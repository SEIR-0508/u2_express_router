## SEIR 0508

# Express Router


In our first Express apps, we used the `app.get` method to define routes and although it worked, the better practice is to:

- Use Express `router` objects to define routes for a particular purpose or dedicated to a certain data resource such as `tasks`, `cats`, `users`.
- Create each `router` in its own module from which it is exported.
- Inside of **server.js** `require` and mount the `router` object in the request pipeline.

> Note:  A **data resource** is a "type" of data/information that applications create, read, update and/or delete (CRUD).

Like our Controllers, Routers are used to encapsulate certain parts of data we want to work with, that will allow us to keep our main folders clean and organized. It also allows people on a team to work on one individual file or folder at a time and push their versions to git, rather than everyone working on one file at once, which as you can imagine, will get messy really quick

### Best Practice Routing Set Up by Express Generator

As an example of using this better approach to routing, let's look at how `express-generator` sets up routing...

First, there's a `routes` folder containing two router modules:

- **AppRouter.js**: Great for defining general purpose routes, e.g., the root route. We can export all our individual routes into an index file and export the index to our Server.js file, like we did with controllers

- **UserRouter.js**: An example of a router dedicated to a _data resource_, in this case, _users_.
Note how routes are defined on those two `router` objects using `router.get()` method call just like we did previously with `app.get()`

Each `router` object has one route defined - compare those two routes, notice the **HTTP methods** and the **paths**?  They're the same - isn't that a problem?  Nope, they're not actually the same because of the way the routers are mounted in **server.js**...


> **IMPORTANT KEY POINT:** The path specified in `app.use` is a **"starts with path"**. It is bound to the paths specified in the router object forming the **actual path**.

### Determining the Actual Path of a Route Defined in a Router Object

Let's say you have a `router` object that defines a route like this:

```js
// routes/brands.js

let express = require('express');
let router = express.Router();

router.get('/brands', (req, res) => {...
```

But wait, having that callback function can get annoying... lets take it up one more notch to do something like this

```js

// routes/brands.js

let express = require('express')
let brandController = require ('../controllers')
let router = express.Router()

router.get('/brands', brandController.getBrands)
router.get('/brands/:id', brandController.getBrandById)


export module = router
```
	
and is mounted like this in our server.js:
	
```js
const brandRouter = require('./routes/brands');

// All routes defined in todosRouter will start with /todos
app.use('/brands', brandRouter);
```



Another example, let's say you have a `router` object that defines a route like this, pulling another funciton from the controller and attaching it to the route. 
We now have (at least) 3 files chained together -> a controller, a router, and our Server.js file. This can get a bit confusing at times, which is why we want to keep everything organized and with semantic names

```js
// routes/products.js

let express = require('express')
let router = express.Router()
let productController = require('../controllers/productController')

router.get('/products', productController.getAllProducts)
```
	
and is mounted like this:
	
```js
const productRouter = require('./routes/calendar');

app.use('/products', calendarRouter);
```



We are going to take all of our Router files and add them to a new file called AppRouter.js, which should look something like this

routes/AppRouter.js
```js

const Router = require('express').Router()
const ProductRouter = require('./ProductRouter')
const BrandRouter = require('./BrandRouter')
Router.use('/products', ProductRouter)
Router.use('/brands', BrandsRouter)

module.exports = Router
```


