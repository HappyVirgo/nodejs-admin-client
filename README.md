## Usage

```js
import KcAdminClient from 'keycloak-admin';
// or
// const KcAdminClient = require('keycloak-admin').default;

// To configure the client, pass an object to override any of these  options:
// {
//   baseUrl: 'http://127.0.0.1:8080/auth',
//   realmName: 'master',
//   requestConfig: {
//     /* Axios request config options https://github.com/axios/axios#request-config */
//   },
// }
const kcAdminClient = new KcAdminClient();

// Authorize with username / password
await kcAdminClient.auth({
  username: 'wwwy3y3',
  password: 'wwwy3y3',
  grantType: 'password',
  clientId: 'admin-cli',
  totp: '123456', // optional Time-based One-time Password if OTP is required in authentication flow
});

// List all users
const users = await kcAdminClient.users.find();

// Override client configuration for all further requests:
kcAdminClient.setConfig({
  realmName: 'another-realm',
});

// This operation will now be performed in 'another-realm' if the user has access.
const groups = await kcAdminClient.groups.find();

// Set a `realm` property to override the realm for only a single operation.
// For example, creating a user in another realm:
await this.kcAdminClient.users.create({
  realm: 'a-third-realm',
  username: 'username',
  email: 'user@example.com',
});
```

To refresh the access token provided by Keycloak, an OpenID client like [panva/node-openid-client](https://github.com/panva/node-openid-client) can be used like this:

```js
import {Issuer} from 'openid-client';

const keycloakIssuer = await Issuer.discover(
  'http://localhost:8080/auth/realms/master',
);

const client = new keycloakIssuer.Client({
  client_id: 'admin-cli', // Same as `clientId` passed to client.auth()
  token_endpoint_auth_method: 'none', // to send only client_id in the header
});

// Use the grant type 'password'
let tokenSet = await client.grant({
  grant_type: 'password',
  username: 'wwwy3y3',
  password: 'wwwy3y3',
});

// Periodically using refresh_token grant flow to get new access token here
setInterval(async () => {
  const refreshToken = tokenSet.refresh_token;
  tokenSet = await client.refresh(refreshToken);
  kcAdminClient.setAccessToken(tokenSet.access_token);
}, 58 * 1000); // 58 seconds
```

In cases where you don't have a refresh token, eg. in a client credentials flow, you can simply call `kcAdminClient.auth` to get a new access token, like this:

```js
const credentials = {
  grantType: 'client_credentials',
  clientId: 'clientId',
  clientSecret: 'some-client-secret-uuid',
};
await kcAdminClient.auth(credentials);

setInterval(() => kcAdminClient.auth(credentials), 58 * 1000); // 58 seconds
```
