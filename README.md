# shopify-token

[![Version npm][npm-shopify-token-badge]][npm-shopify-token]
[![Build Status][ci-shopify-token-badge]][ci-shopify-token]
[![Coverage Status][coverage-shopify-token-badge]][coverage-shopify-token]

This module helps you retrieve an access token for the Shopify REST API. It
provides some convenience methods that can be used when implementing the [OAuth
2.0 flow][shopify-oauth-doc]. No assumptions are made about your server-side
architecture, allowing the module to easily adapt to any setup.

## Install

```
npm install --save shopify-token
```

## API

The module exports a class whose constructor takes an options object.

### `new ShopifyToken(options)`

Creates a new `ShopifyToken` instance.

#### Arguments

- `options` - A plain JavaScript object, e.g. `{ apiKey: 'YOUR_API_KEY' }`.

#### Options

- `apiKey` - Required - A string that specifies the API key of your app.
- `sharedSecret` - Required - A string that specifies the shared secret of your
  app.
- `redirectUri` - Required - A string that specifies the URL where you want to
  redirect the users after they authorize the app.
- `scopes` - Optional - An array of strings or a comma-separated string that
  specifies the list of scopes e.g. `'read_content,read_themes'`. Defaults to
  `'read_content'`.
- `timeout` - Optional - A number that specifies the milliseconds to wait for
  the server to send a response to the HTTPS request initiated by the
  `getAccessToken` method before aborting it. Defaults to 60000, or 1 minute.
- `accessMode` - Optional - A string representing the [API access
  modes][api-access-mode]. Set this option to `'per-user'` to receive an access
  token that respects the user's permission level when making API requests
  (called online access). This is strongly recommended for embedded apps.
  Defaults to offline access mode.

#### Return value

A `ShopifyToken` instance.

#### Exceptions

Throws a `Error` exception if the required options are missing.

#### Example

```js
const ShopifyToken = require('shopify-token');

const shopifyToken = new ShopifyToken({
  sharedSecret: '8ceb18e8ca581aee7cad1ddd3991610b',
  redirectUri: 'http://localhost:8080/callback',
  apiKey: 'e74d25b9a6f2b15f2836c954ea8c1711'
});
```

### `shopifyToken.generateNonce()`

Generates a random nonce.

#### Return value

A string representing the nonce.

#### Example

```js
const nonce = shopifyToken.generateNonce();

console.log(nonce);
// => 212a8b839860d1aefb258aaffcdbd63f
```

### `shopifyToken.generateAuthUrl(shop[, scopes[, nonce[, accessMode]]])`

Builds and returns the authorization URL where you should redirect the user.

#### Arguments

- `shop` - A string that specifies the name of the user's shop.
- `scopes` - An optional array of strings or comma-separated string to specify
  the list of scopes. This allows you to override the default scopes.
- `nonce` - An optional string representing the nonce. If not provided it will
  be generated automatically.
- `accessMode` - An optional string dictating the API access mode. If not
  provided the access mode defined by the `accessMode` constructor option will
  be used.

#### Return value

A string representing the URL where the user should be redirected.

#### Example

```js
const url = shopifyToken.generateAuthUrl('dolciumi');

console.log(url);
// => https://dolciumi.myshopify.com/admin/oauth/authorize?scope=read_content&state=7194ee27dd47ac9efb0ad04e93750e64&redirect_uri=http%3A%2F%2Flocalhost%3A8080%2Fcallback&client_id=e74d25b9a6f2b15f2836c954ea8c1711
```

### `shopifyToken.verifyHmac(query)`

Every request or redirect from Shopify to the client server includes a hmac
parameter that can be used to ensure that it came from Shopify. This method
validates the hmac parameter.

#### Arguments

- `query` - The parsed query string object.

#### Return value

`true` if the hmac is valid, else `false`.

#### Example

```js
const ok = shopifyToken.verifyHmac({
  hmac: 'd1c59b480761bdabf7ee7eb2c09a3d84e71b1d37991bc2872bea8a4c43f8b2b3',
  signature: '184559898f5bbd1301606e7919c6e67f',
  state: 'b77827e928ee8eee614b5808d3276c8a',
  code: '4d732838ad8c22cd1d2dd96f8a403fb7',
  shop: 'dolciumi.myshopify.com',
  timestamp: '1452342558'
});

console.log(ok);
// => true
```

### `shopifyToken.getAccessToken(hostname, code)`

Exchanges the authorization code for a permanent access token.

#### Arguments

- `hostname` - A string that specifies the hostname of the user's shop. e.g.
  `foo.myshopify.com`. You can get this from the `shop` parameter passed by
  Shopify in the confirmation redirect.
- `code` - The authorization Code. You can get this from the `code` parameter
  passed by Shopify in the confirmation redirect.

#### Return value

A `Promise` which gets resolved with an access token and additional data. When
the exchange fails, you can read the HTTPS response status code and body from
the `statusCode` and `responseBody` properties which are added to the error
object.

#### Example

```js
const code = '4d732838ad8c22cd1d2dd96f8a403fb7';
const hostname = 'dolciumi.myshopify.com';

shopifyToken
  .getAccessToken(hostname, code)
  .then((data) => {
    console.log(data);
    // => { access_token: 'f85632530bf277ec9ac6f649fc327f17', scope: 'read_content' }
  })
  .catch((err) => console.err(err));
```

## License

[MIT](LICENSE)

[api-access-mode]: https://shopify.dev/apps/auth/access-modes
[npm-shopify-token-badge]: https://img.shields.io/npm/v/shopify-token.svg
[npm-shopify-token]: https://www.npmjs.com/package/shopify-token
[ci-shopify-token-badge]:
  https://img.shields.io/github/workflow/status/lpinca/shopify-token/CI/master?label=CI
[ci-shopify-token]:
  https://github.com/lpinca/shopify-token/actions?query=workflow%3ACI+branch%3Amaster
[coverage-shopify-token-badge]:
  https://img.shields.io/coveralls/lpinca/shopify-token/master.svg
[coverage-shopify-token]:
  https://coveralls.io/r/lpinca/shopify-token?branch=master
[shopify-oauth-doc]: https://shopify.dev/apps/auth/oauth
