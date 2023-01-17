This is a NodeJS proxy which adds CORS headers to the proxied request.

The url to proxy is literally taken from the path, validated and proxied. The protocol
part of the proxied URI is optional, and defaults to "http". If port 443 is specified,
the protocol defaults to "https".

This package does not put any restrictions on the http methods or headers, except for
cookies. Requesting [user credentials](http://www.w3.org/TR/cors/#user-credentials) is disallowed.
The app can be configured to require a header for proxying a request, for example to avoid
a direct visit from the browser.

## Example

```javascript
// Listen on a specific host via the HOST environment variable
var host = process.env.HOST || '0.0.0.0';
// Listen on a specific port via the PORT environment variable
var port = process.env.PORT || 8080;

var cors_proxy = require('cors-anywhere');
cors_proxy.createServer({
    originWhitelist: [], // Allow all origins
    requireHeader: ['origin', 'x-requested-with'],
    removeHeaders: ['cookie', 'cookie2']
}).listen(port, host, function() {
    console.log('Running CORS Anywhere on ' + host + ':' + port);
});

```
Request examples:

* `http://localhost:8080/http://google.com/` - Google.com with CORS headers
* `http://localhost:8080/google.com` - Same as previous.
* `http://localhost:8080/google.com:443` - Proxies `https://google.com/`
* `http://localhost:8080/` - Shows usage text, as defined in `lib/help.txt`
* `http://localhost:8080/favicon.ico` - Replies 404 Not found

### Client side documentation

A concise summary of the documentation is provided at [lib/help.txt](lib/help.txt).

If you want to automatically enable cross-domain requests when needed, use the following snippet:

```javascript
(function() {
    var cors_api_host = 'localhost:8080';
    var cors_api_url = 'https://' + cors_api_host + '/';
    var slice = [].slice;
    var origin = window.location.protocol + '//' + window.location.host;
    var open = XMLHttpRequest.prototype.open;
    XMLHttpRequest.prototype.open = function() {
        var args = slice.call(arguments);
        var targetOrigin = /^https?:\/\/([^\/]+)/i.exec(args[1]);
        if (targetOrigin && targetOrigin[0].toLowerCase() !== origin &&
            targetOrigin[1] !== cors_api_host) {
            args[1] = cors_api_url + args[1];
        }
        return open.apply(this, args);
    };
})();
```

If you're using jQuery, you can also use the following code **instead of** the previous one:

```javascript
jQuery.ajaxPrefilter(function(options) {
    if (options.crossDomain && jQuery.support.cors) {
        options.url = 'localhost:8080/' + options.url;
    }
});
```
