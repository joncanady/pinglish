# Pinglish

A simple Rack middleware for checking application health. Pinglish
exposes a `/_ping` resource via HTTP `GET`, returning JSON that
conforms to the spec below.

## The Spec

* The application __must__ respond to `GET /_ping` as an HTTP request.

* The request handler __should__ check the health of all services the
  application depends upon, to answer questions like:

  * "Can I query against my MySQL database?"
  * "Can I create/read keys from Redis?"
  * "How many docs are in my ElasticSearch index?"


* The response __must__ return within 29 seconds. This is one second
  less than the default timeout for many monitoring services.

* The response __must__ return an `HTTP 200 OK` status code if all
  health checks pass.

* The response __must__ return an `HTTP 503 SERVICE UNAVAILABLE`
  status code if any health checks fail.

* The response __must__ return an `HTTP 418 I'M A TEAPOT` status code
  if the request asks for any content-type but `application/json`.

* The response __must__ be of Content-Type `application/json;
  charset=UTF-8`.

* The response __must__ be valid JSON. This means even if the
  application is literally shitting itself in the corner, it must
  still return valid JSON.

* The response __must__ contain a `"status"` key set either to `"ok"`
  or `"fail"`.

* The response __must__ contain a `"now"` key set to the current
  server's time in seconds since epoch as a string.

* If the `"status"` key is set to `"fail"`, the response __may__
  contain a `"failures"` key set to an Array of string names
  representing failed checks.

* If the `"status"` key is set to `"fail"`, the response __may__
  contain a `"timeouts"` key set to an Array of string names
  representing checks that exceeded an implementation-specific
  individual timeout.

* The response body __may__ contain any other top-level keys to supply
  additional data about services the application consumes, but all
  values must be strings, arrays of strings, or hashes where both keys
  and values are strings.

### An Example Response

```javascript
{

  // These two keys will always exist.

  "now": "1359055102",
  "status": "fail",

  // This key may only exist when a named check has failed.

  "failures": ["db"],

  // This key may only exist when a named check exceeds its timeout.

  "timeouts": ["really-long-check"],

  // Keys like this may exist to provide extra information about
  // healthy services, like the number of objects in an S3 bucket.

  "s3": "127"
}
```

## The Middleware

FIX: exegesis

```ruby
require "pinglish"

use Pinglish do |ping|

  # A single unnamed check is the simplest possible way to use
  # Pinglish, and you'll probably never want combine it with other
  # named checks. An unnamed check contributes to overall success or
  # failure, but never adds additional data to the response.

  ping.check do
    App.healthy?
  end

  # A named check like this can provide useful summary information
  # when it succeeds. In this case, a top-level "db" key will appear
  # in the response containing the number of items in the database. If
  # a check returns nil, no key will be added to the response.

  ping.check :db do
    App.db.items.size
  end

  # By default, checks time out after one second. You can override
  # this with the :timeout option, but be aware that no combination of
  # checks is ever allowed to exceed the overall 29 second limit.

  ping.check :long, :timeout => 5 do
    App.dawdle
  end

  # Signal check failure by raising an exception.

  ping.check :fails do
    false or raise "Everything's ruined."
  end
end
```

## Contributing

FIX
