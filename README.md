# Passport-Keystone-v3

OpenStack Keystone authentication strategy for [Passport](http://passportjs.org/) and Node.js

This project is a fork of [Passport-Keystone](https://github.com/eddywashere/passport-keystone).  

[Passport-Keystone](https://github.com/eddywashere/passport-keystone) supports up to v2.x . Passport-Keystone-v3 supports up to v3.x.

## Installation

```sh
npm install passport-keystone-v3
```

## Documentation

#### Authentication

The keystone authentication strategy authenticates users using a username and password and optional domainName 
from the POST body.  The strategy requires a `verify` callback, which accepts these credentials and calls `done` 
providing a user that is attached to `req.user`.

```js
passport.use(new KeystoneStrategy({
    authUrl: your.authUrl, // required
    usernameField: 'username', // optional
    passwordField: 'password', // optional
    domainNameField: 'domainName', // optional
    region: your.region, // optional
    tenantId: your.tenantId // optional
  },
  function(user, done) {
    var user = {
      id: identity.user.id,
      token: identity.token.id,
      username: identity.user.name,
      serviceCatalog: identity.raw.access.serviceCatalog
    };
    return done(null, user);
  }
));
```

#### Need to set session expiration to token expiration?

The following example uses `passReqToCallback` to send the `req` object to next callback, where session expiration 
can be configured.

```js
passport.use(new KeystoneStrategy({
    authUrl: your.authUrl, // required
    usernameField: 'username', // optional
    passwordField: 'password' // optional
    domainNameField: 'domainName', // optional
    region: your.region, // optional
    tenantId: your.tenantId // optional
    passReqToCallback : true // allows us to interact with req object
}, function(req, identity, done) {
  if (!req.user) {
    var user = {
        id: identity.user.id,
        token: identity.token.id,
        username: identity.user.name,
        serviceCatalog: identity.raw.access.serviceCatalog
    };

    // Set session expiration to token expiration
    req.session.cookie.expires = Date.parse(identity.token.expires) - Date.now();

    done(null, user);
  } else {
      // user already exists
      var user = req.user; // pull the user out of the session
      return done(null, user);
  }
}));
```

#### Authenticate Requests

Use `passport.authenticate()`, specifying the `'keystone'` strategy, to
authenticate requests.

```js
app.post('/login',
  passport.authenticate('keystone', { failureRedirect: '/login' }),
  function(req, res) {
    res.redirect('/');
  }
);
```

Example form markup

```html
<form action="/login" method="post">
  <label>Username:</label>
  <input type="text" name="username"/><br/>
  <label>Password:</label>
  <input type="password" name="password"/>
  <input type="submit" value="Submit"/>
</form>
```

Example form markup for Keystone V3.x with domain support

```html
<form action="/login" method="post">
  <label>Username:</label>
  <input type="text" name="username"/><br/>
  <label>Password:</label>
  <input type="password" name="password"/>
  <label>Domain Name:</label>
  <input type="text" name="domainName"/>
  <input type="submit" value="Submit"/>
</form>
```

Example request via curl

```sh
curl -v -d "username=bob&password=secret" http://127.0.0.1:3000/login
```

Example request via curl for Keystone V3.x with domain support

```sh
curl -v -d "username=bob&password=secret&domainName=my_domain" http://127.0.0.1:3000/login
```

Checkout [Passportjs.org](http://passportjs.org/guide/authenticate/) for more authentication examples.
