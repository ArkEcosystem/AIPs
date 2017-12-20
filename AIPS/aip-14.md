```
  AIP: 14
  Title: RESTful API
  Authors: Brian Faust <hello@brianfaust.me>
  Status: Draft
  Type: Standards Track
  Created: 2017-12-20
  Last Update: 2017-12-20
```

Abstract
========

This AIP proposes the implementation of a RESTful API that will provide an architecture that will make future development easier and require less changes on the client side.

Motivation
==========

The current implementation of the API is just a collection of endpoints without following any standards and provides poor navigation over large datasets.

As ARK continues to grow, more and more clients will start to use the API and there will come issues up that make working with the API tedious and require workarounds such as running your own version of the core with a modified API.

Rationale
=========

### Versioning

The API should be versioned based on a header to avoid any changes to the public endpoints (URLs). Using **Restify** this can be achieved by using an `Accept-Version` header.

### Resource URLs

When building a RESTful API a URL should only be concerned about the Resource it is responsible for and any child resources if nested URLs are going to be used. Also the URLs should be as descriptive as possible to make it easy to create logical connections.

This means instead of `GET /accounts/getBalance?id=123` a URL should look like `GET /wallets/123/balance` or `GET /wallets/123?columns=balance`.

### Pagination

Making it easy to paginate large datasets is a must have for modern applications. Using **Sequelize** this process can be simplified and is easily extendable.

### Status Codes

The use of HTTP Status Codes as indicators for response statuses will make it possible to remove the `success` field which indicates the status of a response. This will allow to change the response format in the future without breaking any checks if a response has been successful on the client side.

Specifications
==============

### Versioning

This can be solved quite easily using http://restify.com/docs/home/#versioned-routes. This will allow to define routes with versions and serve them accordingly.

```js
const restify = require('restify');
const server = restify.createServer();

class WalletsVersionOne
{
    function show(req, res, next) {
        res.send(Wallet.find(req.body.id));

        return next();
    }
}

class WalletsVersionTwo
{
    function show(req, res, next) {
        res.send(Wallet.findOrFail(req.body.id));

        return next();
    }
}

server.get({ path: '/wallets/:id', version: '1.0.0'}, WalletsVersionOne.show);
server.get({ path: '/wallets/:id', version: '2.0.0'}, WalletsVersionTwo.show);

server.listen(8080);
```

Requests then could be send like this to get data from different versions of the API.

```sh
# use 1.0.0 to 1.9.9
$ curl -s -H 'accept-version: ~1' http://localhost:4001/wallets/some-wallet-address

# use 2.0.0 to 2.9.9
$ curl -s -H 'accept-version: ~2' http://localhost:4001/wallets/some-wallet-address

# will return an exception that informs the client that 3.0.0 is undefined
$ curl -s -H 'accept-version: ~3' http://localhost:4001/wallets/some-wallet-address
```

### Resource URLs

As previously mentioned URLs in a RESTful API shouldn't be concerned about anything but the Resource they are responsible for and be kept as simple as possible.

```
GET /api/wallets
GET /api/wallets/12345
POST /api/wallets
PATCH /api/wallets/1234
DELETE /api/wallets/1234
```

Those URLs together with the HTTP verbs used are descriptive and make clear what is being accessed. An example of a refactor could look like the following.

`GET /accounts`
`GET /wallets`

`GET /accounts/getBalance?id=123`
`GET /wallets/123?columns=balance`

`GET /accounts/getPublicKey?id=123`
`GET /wallets/123?columns=publicKey`

Resources that have relationships to the resource that is being accessed should be accessible as nested resources as this allows for easy access without causing any overhead on the client side.

`GET /wallets/123/blocks`
`GET /wallets/123/transactions`

### Pagination

Handling pagination of large datasets can be a difficult task and result in major performance issues if done the wrong way. Using http://docs.sequelizejs.com/ can ease this process and is easily extendable.

```js
const restify = require('restify');
const server = restify.createServer();

class Wallets
{
    function index(req, res, next) {
        let page = req.params.page;
        let perPage = req.params.perPage;

        let wallets = Wallet.findAll({ offset: page * perPage, limit: perPage });

        res.send({
            data: wallets,
            links: {
                first: req.baseUrl + '?page=' + 1,
                last: req.baseUrl + '?page=' + Math.max(wallets.length / perPage),
                prev: req.baseUrl + '?page=' + (page - 1), // ignore on first page
                next: req.baseUrl + '?page=' + (page + 1), // ignore on last page
            }
            meta: {
                path: req.baseUrl,
                current_page: page,
                first_page: 1,
                last_page: Math.max(wallets.length / perPage),
                per_page: perPage,
                total: wallets.length,
            }
        });

        return next();
    }
}

server.get({ path: '/accounts', version: '2.0.0'}, Wallets.index);

server.listen(8080);
```

### Status Codes

`200` **OK**
- This status should be returned if no warnings or errors occurred.

`400` **Bad Request**
- This status should be returned if an unknown error occurred that is the result of bad data from the client side.

`401` **Unauthorized**
- This status should be returned if the secrets provided are invalid.

`403` **Forbidden**
- This status should be returned if someone is authenticated but not authorized to access an API endpoint.

`404` **Not Found**
- This status should be returned if a resource could not be found.

`405` **Method Not Allowed**
- This status should be returned if an invalid HTTP method is used, e.g. sending a GET request to a POST endpoint.

`422` **Unprocessable Entity**
- This status should be returned if any validation errors occurr like missing or malformed data.

`429` **Too Many Requests**
- This status should be returned if too many requests are send within a short period of time, basically throttling.
