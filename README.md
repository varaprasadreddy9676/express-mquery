express-mquery
====================

[![Build Status](https://travis-ci.org/lykmapipo/express-mquery.svg?branch=master)](https://travis-ci.org/lykmapipo/express-mquery)

Expose [mongoose](https://github.com/Automattic/mongoose) query API through HTTP request with partial support of [json-api](http://jsonapi.org/): 
 - [sparse-fieldsets](http://jsonapi.org/format/#fetching-sparse-fieldsets)
 - [sorting](http://jsonapi.org/format/#fetching-sorting)
 - [pagination](http://jsonapi.org/format/#fetching-pagination)
 - [filtering](http://jsonapi.org/format/#fetching-filtering)
 - [includes](http://jsonapi.org/format/#fetching-includes)

*Note: Due to refactoring, you must install [mongoose-rest-actions](https://github.com/lykmapipo/mongoose-rest-actions) to have back mongoose plugin*

## Installation
```js
$ npm install --save express-mquery mongoose-rest-actions
```

## Usage
```js
//plugin rest actions plugin
const mongoose = require('mongoose');
const actions = require('mongoose-rest-actions');
mongoose.plugin(actions);

//add express middleware
const expess = require('express');
const mquery = require('express-mquery');

const app = express();
app.use(mquery({ limit: 10, maxLimit: 50 }));

...

app.get('/users', function(request, response, next) {
  
  //obtain request.mquery
  const options = request.mquery;

  User
    .get(options, function(error, results) {
      ...handle error or reply
    });

});
```
## Structure
Once parse, `express-mquery` will extend `http request` with `mquery` field
with the structure below

```js
GET /invoices?fields=number,amount&filter[name]=Bob&filter[amount][$gte]=1200&include=customer,items&fields[customer]=name,number&fields[items]=name,price&page[number]=1&page[size]=10&sort[number]=1&sort[amount]=-1

{
  filter: { name: "Bob", amount: { $gte: 1200 } },
  paginate: { limit: 10, skip: 0, page: 1 },
  populate: [
  	{ path: "customer", select: { name: 1, number: 1 } },
  	{ path: "items", select: { name: 1, price: 1 } }
  ],
  select: { number: 1, amount: 1 },
  sort: { number: 1, amount: -1 }
}

```


## Querying
All the following parameters `(sort, page, skip, limit, query, populate, select)` support the entire mongoose feature set.

>When passing values as objects or arrays in URLs, they must be valid JSON

### Sort or Order
```js
GET /customers?sort=name
GET /customers?sort=-name
GET /customers?sort={"name":1}
GET /customers?sort={"name":1, "email":-1}

or

GET /customers?sort=name
GET /customers?sort=-name
GET /customers?sort[name]=1&sort[email]=-1
```

### Page
```js
GET /customers?page=1
GET /customers?page=1&limit=10
GET /customers?page[number]=1&page[size]=10
```

### Skip
```js
GET /customers?skip=10
```

### Limit
Only overrides `maximum limit option set by the plugin` if the queried limit is lower
```js
GET /customers?limit=10
```

### Query or Filters
Supports all [mongodb operators](https://docs.mongodb.com/manual/reference/operator/query/) `($regex, $gt, $gte, $lt, $lte, $ne, etc.)`

```js
GET /customers?query={"name":"Bob"}
GET /customers?query={"name":{"$regex":"/Bo$/"}}
GET /customers?query={"age":{"$gt":12}}
GET /customers?query={"age":{"$gte":12}}

or

GET /customers?filter[name]=Bob
GET /customers?filter[name][$regex]="/Bo$/"
GET /customers?filter[age][$gt]=12
GET /customers?filter[age][$gte]=12
```

### Populate or Include
Works with create, read and update operations

```js
GET /invoices?populate=customer
GET /invoices?populate={"path":"customer"}
GET /invoices?populate=[{"path":"customer"},{"path":"products"}]

or

GET /invoices?include=customer
GET /invoices?include[customer]=name,number&includes[items]=name,price
GET /invoices?include=customer,items&fields[customer]=name,number&fields[items]=name,price
```

### Select or Fields
`_id` is always returned unless explicitely excluded

```js
GET /customers?select=name
GET /customers?select=-name
GET /customers?select={"name":1}
GET /customers?select={"name":0}

or

GET /customers?fields=name
GET /customers?fields=-name
GET /customers?fields=name,email
GET /invoices?include=customer&fields[customer]=name
```

## Testing

* Clone this repository

* Install `grunt-cli` global

```sh
$ npm install -g grunt-cli
```

* Install all development dependencies

```sh
$ npm install
```

* Then run test

```sh
$ npm test
```

## Contribute

Fork this repo and push in your ideas. Do not forget to add a bit of test(s) of what value you adding.

## Licence

Copyright (c) 2015 lykmapipo & Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE. 
