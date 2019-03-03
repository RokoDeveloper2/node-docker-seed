## NodeJS Seed Project

### Build status
[![CircleCI](https://circleci.com/gh/RapidAPI/nodejs-service-seed.svg?style=svg&circle-token=1c9876a5d919166120a1a4c38f8cef357f93a452)](https://circleci.com/gh/RapidAPI/nodejs-service-seed)

### Contains:
- [Express](https://github.com/expressjs/express)
 - [Body-Parser](https://github.com/expressjs/body-parser)
 - [ESLint](https://github.com/eslint/eslint)
 - [Winston](https://github.com/winstonjs/winston)
 - [NewRelic](https://github.com/newrelic/node-newrelic)
 - [fs](https://nodejs.org/docs/v0.3.1/api/fs.html)
 - [Config](https://github.com/lorenwest/node-config)
 - [Pre-commit](https://github.com/observing/pre-commit)
 - [Joi](https://github.com/hapijs/joi)
 
### Health check (For load balancers, etc)
GET HOST:PORT/ping should return 200 { payload: "pong" }

### Steps after clone:
1. Change the service name in config/default.json. e.g. "ServiceName": "Seed" -> "ServiceName": "Discussions"

### Routes
For every directory in the `modules` directory there must be a .routes.js file with the same name, for example: in `users` directory you would have `users.routes.js`. This file exports a function that recieves the Express object on which you would define your routes.
When defining a route, you can use any of the following middlewares: `validator`, `filter`, and `endpoint`. Your route should look something like this:
```javascript
module.exports = app => {
  app.get("/user/:username", filters, validator(getUser.validations), endpoint(getUser.endpoint));
};
```

### Validator middleware
The validator middleware allows you to easily check body, route, and query parameters and will return error codes in a standardized manner in case any checks failed.
Validations use the [Joi](https://github.com/hapijs/joi) library, and are defined so:
```javascript
module.exports.validations = {
  params: {
    num: Joi.number().max(10).required()
  },
  query: {
    float: Joi.boolean()
  },
  body: {
    data: Joi.object().keys({
      name: Joi.string().required(),
      age: Joi.number()
    })
  }
};
```

### Endpoint middleware
The endpoint middleware is the component responsible for running the logic of your endpoint and returning results in a standardized manner. Your endpoint must return an object with a status code (usually 200, 201, or 204 for GET, POST, and PUT/DELETE respectively) and optionally a `data` member with return data. In case of errors, your endpoint should `throw` an exception, either one of the provided ones or a custom one.
Your endpoint function receives one argument, the Express `request` object, and should be defined so:
```javascript
module.exports.endpoint = async req => {
  const { username } = req.params;
  const where = {
    username
  };
  const data = await Database.findOne({ where });
  if (!data) throw new NotFound(`Could not find ${username} in DB`);
  return { status: 200, data };
};
```

### Filters middleware
The filters middleware allows receiving filters and other data-related queries. This will create a `filters` object on the `request` object that your endpoint receives from the Endpoint middleware, which must then be passed onto your ORM to deal with accordingly.
The following filters are supported:
* `limit` - the amount of items to return from a `GET` request
* `order` - the order in which to query the DB
* `page` - the page offset
* `query` - a JSON object to create a query to the DB

### Changelog
- **Jul. 13, 2017** - Initial commit
- **Nov. 17, 2017** - Extended endpoints, validations, and filters middlewares
- **Nov. 18, 2017** - Add CircleCI support
