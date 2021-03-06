# Supertest sessions

Session wrapper around supertest.

[![Build
Status](https://travis-ci.org/rjz/supertest-session.svg?branch=master)](https://travis-ci.org/rjz/supertest-session)
[![Coverage
Status](https://coveralls.io/repos/rjz/supertest-session/badge.png)](https://coveralls.io/r/rjz/supertest-session)

References:

  * https://gist.github.com/joaoneto/5152248
  * https://github.com/visionmedia/supertest/issues/46
  * https://github.com/visionmedia/supertest/issues/26

## Test

    $ npm test

## Usage

Require `supertest-session` and pass in the test application:

```js
var session = require('supertest-session');
var myApp = require('../../path/to/app');

var testSession = null;

beforeEach(function () {
  testSession = session(myApp);
});
```

And set some expectations:

```js
it('should fail accessing a restricted page', function (done) {
  testSession.get('/restricted')
    .expect(401)
    .end(done)
});

it('should sign in', function (done) {
  testSession.post('/signin')
    .send({ username: 'foo', password: 'password' })
    .expect(200)
    .end(done);
});
```

You can set preconditions:

```js
describe('after authenticating session', function () {

  var authenticatedSession;

  beforeEach(function (done) {
    testSession.post('/signin')
      .send({ username: 'foo', password: 'password' })
      .expect(200)
      .end(function (err) {
        if (err) return done(err);
        authenticatedSession = testSession;
        return done();
      });
  });

  it('should get a restricted page', function (done) {
    authenticatedSession.get('/restricted')
      .expect(200)
      .end(done)
  });

});

```

### Accessing session data

The cookies attached to the session may be retrieved from `session.cookies` at
any time, for instance to inspect the contents of the current session in an
external store.

```js
it('should set session details correctly', function (done) {
  var sessionCookie = _.find(testSession.cookies, function (cookie) {
    return cookie.name === connect.sid;
  });

  memcached.get(sessionCookie.value, function (err, session) {
    session.user.name.should.eq('Foobar');
    done();
  });
});
```

### Request hooks

By default, supertest-session authenticates using session cookies. If your app
uses a custom strategy to restore sessions, you can provide `before` and `after`
hooks to adjust the request and inspect the response:

```js
var testSession = session(myApp, {
  before: function (req) {
    req.set('authorization', 'Basic aGVsbG86d29ybGQK');
  }
});
```

## License

MIT

