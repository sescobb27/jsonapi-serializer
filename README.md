# JSON API Serializer
[![Build Status](https://travis-ci.org/SeyZ/jsonapi-serializer.svg?branch=master)](https://travis-ci.org/SeyZ/jsonapi-serializer)

A Node.js framework agnostic library for serializing your data to [JSON
API](http://jsonapi.org) (1.0 compliant).

## Installation
`$ npm install jsonapi-serializer`

## Documentation

**JSONAPISerializer(type, data, opts)** serializes the *data* (can be an object or an array) following the rules defined in *opts*.

- type: The resource type.
- data: An object to serialize.
- opts
    - *attributes*: An array of attributes to show. You can define an attribute as an option if you want to define some relationships (included or not).
        - *ref*: If present, it's considered as a relationships.
        - *included*: Consider the relationships as [compound document](http://jsonapi.org/format/#document-compound-documents). Default: true.
        - *attributes*: An array of attributes to show.
        - *topLevelLinks*: An object that describes the top-level links. Values can be *string* or a *function* (see examples below)
        - *dataLinks*: An object that describes the links inside data. Values can be *string* or a *function* (see examples below)
        - *relationshipLinks*: An object that describes the links inside relationships. Values can be *string* or a *function* (see examples below)
        - *ignoreRelationshipData*: Do not include the `data` key inside the relationship. Default: false.
        - *keyForAttribute*: A function or string to customize attributes. Functions are passed the attribute as a single argument and expect a string to be returned. Strings are aliases for inbuilt functions for common case conversions. Options include:
          - dash-case (default)
          - lisp-case
          - spinal-case
          - kebab-case
          - underscore_case
          - snake_case
          - CamelCase
          - camelCase
        - *pluralizeType*: A boolean to indicate if the type must be pluralized or not. Default: true.
        - *typeForAttribute*: A function that maps the attribute (passed as an argument) to the type you want to override. Option *pluralizeType* ignored if set.
        - *meta*: An object to include non-standard meta-information.

## Examples

- [Express example](https://github.com/SeyZ/jsonapi-serializer/tree/master/examples/express)
- [Simple](#simple-usage)
- [Nested resource](#nested-resource)
- [Compound document](#compound-document)

<a name="simple-usage"/>
### Simple usage

```javascript
// Sample data object
var data = [{
    id: 1,
    firstName: 'Sandro',
    lastName: 'Munda'
  },{
    id: 2,
    firstName: 'John',
    lastName: 'Doe'
  }];
```

```javascript
var JSONAPISerializer = require('jsonapi-serializer').Serializer;

var users =new JSONAPISerializer('users', data, {
  topLevelLinks: { self: 'http://localhost:3000/api/users' },
  dataLinks: {
    self: function (user) {
      return 'http://localhost:3000/api/users/' + user.id
    }
  },
  attributes: ['firstName', 'lastName']
});

// `users` here are JSON API compliant.
```

The result will be something like:

```javascript
{
  "links": {
    "self": "http://localhost:3000/api/users"
  },
  "data": [{
    "type": "users",
    "id": "1",
    "attributes": {
      "first-name": "Sandro",
      "last-name": "Munda"
    },
    "links": "http://localhost:3000/api/users/1"
  }, {
    "type": "users",
    "id": "2",
    "attributes": {
      "first-name": "John",
      "last-name": "Doe"
    },
    "links": "http://localhost:3000/api/users/2"
  }]
}
```

<a name="nested-resource"/>
### Nested resource
```javascript
var JSONAPISerializer = require('jsonapi-serializer');

var users = new JSONAPISerializer('users', data, {
  topLevelLinks: { self: 'http://localhost:3000/api/users' },
  attributes: ['firstName', 'lastName', 'address'],
  address: {
    attributes: ['addressLine1', 'zipCode', 'city']
  }
});

// `users` here are JSON API compliant.
```

The result will be something like:

```javascript
{
  "links": {
    "self": "http://localhost:3000/api/users"
  },
  "data": [{
    "type": "users",
    "id": "1",
    "attributes": {
      "first-name": "Sandro",
      "last-name": "Munda",
      "address": {
        "address-line1": "630 Central Avenue",
        "zip-code": 24012,
        "city": "Roanoke"
      }
    }
  }, {
    "type": "users",
    "id": "2",
    "attributes": {
      "first-name": "John",
      "last-name": "Doe",
      "address": {
        "address-line1": "400 State Street",
        "zip-code": 33702,
        "city": "Saint Petersburg"
      }
    }
  }]
}
```

<a name="compound-document"/>
### Compound document

```javascript
var JSONAPISerializer = require('jsonapi-serializer');

var users = new JSONAPISerializer('users', data, {
  topLevelLinks: { self: 'http://localhost:3000/api/users' },
  attributes: ['firstName', 'lastName', 'books'],
  books: {
    ref: '_id',
    attributes: ['title', 'isbn'],
    relationshipLinks: {
      "self": "http://example.com/relationships/books",
      "related": "http://example.com/books"
    },
    includedLinks: {
      self: function (dataSet, book) {
        return 'http://example.com/books/' + book.id;
      }
    }
  }
});

// `users` here are JSON API compliant.
```

The result will be something like:

```javascript
{
  "links": {
    "self": "http://localhost:3000/api/users"
  },
  "data": [{
    "type": "users",
    "id": "1",
    "attributes": {
      "first-name": "Sandro",
      "last-name": "Munda"
    },
    "relationships": {
      "books": {
        "data": [
          { "type": "books", "id": "1" },
          { "type": "books", "id": "2" }
        ],
        "links": {
          "self": "http://example.com/relationships/books",
          "related": "http://example.com/books"
        }
      }
    }
  }, {
    "type": "users",
    "id": "2",
    "attributes": {
      "first-name": "John",
      "last-name": "Doe"
    },
    "relationships": {
      "books": {
        "data": [
          { "type": "books", "id": "3" }
        ],
        "links": {
          "self": "http://example.com/relationships/books",
          "related": "http://example.com/books"
        }
      }
    }
  }],
  "included": [{
  	"type": "books",
  	"id": "1",
  	"attributes": {
  	  "title": "La Vida Estilista",
  	  "isbn": "9992266589"
  	},
    "links": {
      "self": "http://example.com/books/1"
    }
  }, {
   "type": "books",
   "id": "2",
   "attributes": {
  	  "title": "La Maria Cebra",
  	  "isbn": "9992264446"
  	},
    "links": {
     "self": "http://example.com/books/2"
    }
  }, {
   "type": "books",
   "id": "3",
   "attributes": {
  	  "title": "El Salero Cangrejo",
  	  "isbn": "9992209739"
  	},
    "links": {
      "self": "http://example.com/books/3"
    }
  }]
}
```

**JSONAPIDeSerializer(collectionName, payload)** deserializes a JSON API *payload*
(e.g req.body or res.body) into *collectionName* it can be either `{}` or
an Object "Class".

- collectionName: {} or Object ("Class")
- payload: JSON API compliant payload

JSON API Payload
```javascript
```

Result
```javascript
```

Example with Express Server and Mongoose

```javascript
var express = require('express');
var bodyParser = require('body-parser');
var JSONAPIDeSerializer = require('jsonapi-serializer').DeSerializer;
var _ = require('lodash');

// parse application/json
app.use(bodyParser.json());

// parse application/vnd.api+json as json
app.use(bodyParser.json({
  type: 'application/vnd.api+json'
}));

/**
 * JSON API Content-Type HEADER
 */
app.use((req, res, next) => {
  res.set('Content-Type', 'application/vnd.api+json');
  next();
});

var orders = require('./lib/orders');

// JSON API deserializer as express middleware
app.use((req, res, next) => {
  // If request Content-Type is plain JSON do not serialize
  // If request does not have body (e.g) GET or req.body = {} do not serialize
  // If request body is not JSON API compliant do not serialize (e.g) req.body is not empty but not inside data
  if (req.headers['Content-Type'] === 'application/json' ||
    _.isEmpty(req.body) ||
    _.isEmpty(req.body.data)) {
    return next();
  }
  var model = new JSONAPIDeSerializer({}, req.body);
  // Only pick not (null or undenfined) attributes
  model = _.pick(model, (attr) => {
    return attr !== null && attr !== undefined;
  });
  req.model = model;
  next();
});

// Resources
app.use('/orders', orders);
```

Normalize JSON API relationships into Mongoose relationships

```javascript
var _ = require('lodash');

// relationships comes in { plan: { id: '1-shirt' } }
// but because we are using Mongoose, id doesn't allow us to use plain JS objects
// so we need to extract ids
function normalize(relationships) {
  var normalizedData = {};
  relationships = relationships || {};
  Object.keys(relationships).forEach((key) => {
    var relationData = relationships[key].data;
    if (!relationData) {
      return;
    }
    if (_.isArray(relationData)) {
      normalizedData[key] = relationData.map((relationObject) => relationObject.id);
    } else if (relationData.id) {
      normalizedData[key] = relationData.id;
    }
  });
  return normalizedData;
}

module.exports.normalize = normalize;
```

Order Schema
```javascript
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

var OrderSchema = new Schema({
  customer: { type: Schema.Types.ObjectId, ref: 'Customer' },
  address: { type: Schema.Types.ObjectId, ref: 'Address'},
  season: { type: String },
  details: [{ type: Schema.Types.ObjectId, ref: 'OrderDetail' }],
  createdAt: { type: Date, default: Date.now() },
  updatedAt: { type: Date, default: Date.now() }
});

```

Order's Route
```javascript
var Express = require('express');
var Order = require('ec-domain').Order;
var Serializer = require('../serializers');
var OrdersSerializer = require('../serializers/orders_serializer');

var app = new Express();

/**
 * POST
 */
.post((req, res, next) => {
  var newOrder = req.model; // => From Express Middleware
  // Then normalize JSON API relationships
  _.merge(newOrder, Serializer.normalize(req.body.data.relationships));

  // Then create Order schema
  return Order.create(newOrder).then((order) => {
    debug('order created', order.id);
    // On success serialize created order
    const jsonapi = new OrdersSerializer(order.toObject()).serialize();
    return res.status(201).send(jsonapi);
  }, next);
});

module.exports = app;
```

# License

[MIT](https://github.com/SeyZ/jsonapi-serializer/blob/master/LICENSE)
