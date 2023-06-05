## SEIR 0508

# Express Router


In our first Express apps, we used the `app.get` method to define routes and although it worked, the better practice is to:

- Use Express `router` objects to define routes for a particular purpose or dedicated to a certain data resource such as `tasks`, `cats`, `users`.
- Create each `router` in its own module from which it is exported.
- Inside of **server.js** `require` and mount the `router` object in the request pipeline.

> Note:  A **data resource** is a "type" of data/information that applications create, read, update and/or delete (CRUD).

Like our Controllers, Routers are used to encapsulate certain parts of data we want to work with, that will allow us to keep our main folders clean and organized. It also allows people on a team to work on one individual file or folder at a time and push their versions to git, rather than everyone working on one file at once, which as you can imagine, will get messy really quick


If you do not have your Brands and Products shaped by Controllers yet, here is what the code should look like...


```js
controllers/brandController.js

const { Brand } = require('../models')
const brandSchema = require('../models/brand')

const getBrands = async (req, res)=> {
    const brands = await Brand.find({})
    res.json(brands)
}

const getBrandById = async (req,res) => {
    try{
    const { id } = req.params
    const brand = await Brand.findById(id)
    if(!brand) throw Error('brand not found')
    res.json(brand)
    }catch (e){
        console.log(e)
        res.send('brand not found')
    }
}

module.exports = {
    getBrands,
    getBrandById
}

```

```js
controllers.productController.js

const { Product } = require('../models')
const productSchema = require('../models/product')

const getProducts = async (req, res)=> {
    const products = await Product.find({})
    res.json(product)
}

const getProductById = async (req,res) => {
    try{
    const { id } = req.params
    const product = await Product.findById(id)
    if(!product) throw Error(product not found')
    res.json(product)
    }catch (e){
        console.log(e)
        res.send('product not found')
    }
}

module.exports = {
    getProducts,
    getProductById
}



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
const Router = require('express').Router()
const controller = require('../controllers/brandController')

//index and show routes for our model
Router.get('/brands', controller.getBrands)
Router.get('/brands/:id', controller.getBrandById)

//we may not have these routes yet, but this is how it would look with full CRUD on a model
Router.post('/brands', controller.createBrand)
Router.put('/brands/:brand_id', controller.updateBrand)
Router.delete('/brands/:brand_id', controller.deleteBrand)

module.exports = Router
```
	
and is mounted like this in our server.js:
	
```js
const brandRouter = require('./routes/brands');

// All routes defined in todosRouter will start with /todos
app.use('/brands', brandRouter);
```

Note: We will be editing this file once we get past the next step, so hold on, it won't all work yet!

### You Do - 10 Minutes
Now that we have our Brands router set up, lets do the same for our Products Router. If we don't have the full CRUD functionality for them yet, thats okay, make sure to recognize the patterns that exist in each block!


```js
// routes/products.js

let express = require('express')
let router = express.Router()
let productController = ...

//our Index route
router.get('/products', ...)

//create a Show route for this data model
```

### Attaching our Routes into our App Router
	
We are going to take all of our Router files and add them to a new file called AppRouter.js, which should look something like this

routes/AppRouter.js
```js


const express = require('express');
//using this thing called the Router, which is build into express
const router = express.Router()
//importing our two individual router files
const ProductRouter = require('./productRouter')
const BrandRouter = require('./brandRouter')

//setting up our basic routes
Router.use('/products', ProductRouter)
Router.use('/brands', BrandsRouter)

module.exports = Router
```


### Getting our Server.js file to work

We now need to require our AppRouter into our Server.js file, and Use these routes, which will look something like this!

server.js
```js
const express = require('express')
const app = express()
const cors = require('cors')

//requiring our AppRouter which holds the individual routes
const AppRouter = require('./routes/AppRouter')

const PORT = process.env.PORT || 3001

app.use(cors())
app.use(express.json())
app.use(express.urlencoded({ extended: true }))

//creating a landing page
app.get('/', (req, res) => res.json({ message: 'Server Works' }))
//creating our API routes. It is common to see /api/ before these routes so that you know which part of your page you are working in, and so that you can have other pages (Login, Register...) that are completely removed from these data points
app.use('/api', AppRouter)
app.listen(PORT, () => console.log(`Server Started On Port: ${PORT}`))

```

But wait! Our routes are now redudant! To see our products you'd have to go to `localhost:3001/api/products/products`
Lets edit our route files and remove those data names, so our routes are much cleaner

Tip - List and Create routes go to our Index routes, Show, Update, and Delete routes go to the Display routes, the ones linked by an ID

### Refactoring BrandsRouter


```js


// routes/brands.js
const express = require('express');
const Router = express.Router()
const controller = require('../controllers/brandController')

//index and show routes for our model
Router.get('/', controller.getBrands)
Router.get('//:id', controller.getBrandsById)

Router.post('/'.controller.createBrand)
Router.put('/:id', controller.updateBrand)
Router.delete('/id', controller.deleteBrand)

module.exports = Router

```

Lets do the same thing for Products now. Again, which controllers will connect the Index route, and which will go to the Show routes by :id?



You should now be able to see your data at:
- localhost:3001/api/products
- localhost:3001/api/products/:id
- localhost:3001/api/brands
- localhost:3001/api/brands/:id

But look how much cleaner your Express ecosystem is, and how much better it will be to work with a team on these!
