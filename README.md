# RequireJS - Base64

A [RequireJS](http://requirejs.org)/AMD loader plugin for loading resources as base64 strings.

## Latest release

The latest release is always available from [the "latest" tag](https://raw.githubusercontent.com/d3x7r0/requirejs-base64-plugin/latest/base64.js).

It can also be installed using [bower](http://bower.io/):

    bower install requirejs-base64-plugin

## Usage

Sometimes it's useful to be able to load resources like images as base64 strings, 
especially when bundling modules using [r.js](https://github.com/jrburke/r.js/)

The base64 AMD loader plugin can help with this issue. It will load any file as 
a base64 string if the base64! prefix is used for a dependency. Download the plugin 
and put it in the app's [baseUrl](http://requirejs.org/docs/api.html#config-baseUrl)
directory (or use the [paths config](http://requirejs.org/docs/api.html#config-paths) to place it in other areas).

You can specify a base64 resource as a dependency like so:

```javascript
require(["some/module", "base64!some/file.png", "text!some/file.jpg"],
    function(module, png, jpg) {
        // the png and jpg variables will be
        // a base64 string reprensentation of
        // the original files
    }
);
```

Notice the .png and .jpg suffixes to specify the extension of the file. The
"some/file" part of the path will be resolved according to normal module name
resolution: it will use the **baseUrl** and **paths** [configuration
options](http://requirejs.org/docs/api.html#config) to map that name to a path.

The files are loaded via asynchronous XMLHttpRequest (XHR) calls, so you
can only fetch files from the same domain as the web page (see **XHR
restrictions** below).

However, [the RequireJS optimizer](http://requirejs.org/docs/optimization.html)
will inline any base64! references with the actual file contents into the
modules, so after a build, the modules that have base64! dependencies can be used
from other domains.

## Configuration

### XHR restrictions

The plugin works by using XMLHttpRequest (XHR) to fetch the binary data for the
resources it handles.

However, XHR calls have some restrictions, due to browser/web security policies:

1) Many browsers do not allow file:// access to just any file. You are better
off serving the application from a local web server than using local file://
URLs. You will likely run into trouble otherwise.

2) There are restrictions for using XHR to access files on another web domain.
While CORS can help enable the server for cross-domain access, doing so must
be done with care (in particular if you also host an API from that domain),
and not all browsers support CORS.

So if the text plugin determines that the request for the resource is on another
domain, it will try to access a ".js" version of the resource by using a
script tag. Script tag GET requests are allowed across domains. The .js version
of the resource should just be a script with a define() call in it that returns
a string for the module value.

Example: if the resource is 'base64!example.png' and that resolves to a path
on another web domain, the text plugin will do a script tag load for
'example.png.js'.

The [requirejs optimizer](http://requirejs.org/docs/optimization.html) will
generate these '.js' versions of the text resources if you set this in the
build profile:

    optimizeAllPluginResources: true

In some cases, you may want the text plugin to not try the .js resource, maybe
because you have configured CORS on the other server, and you know that only
browsers that support CORS will be used. In that case you can use the
[module config](http://requirejs.org/docs/api.html#config-moduleconfig)
(requires RequireJS 2+) to override some of the basic logic the plugin uses to
determine if the .js file should be requested:

```javascript
requirejs.config({
    config: {
        text: {
            useXhr: function (url, protocol, hostname, port) {
                //Override function for determining if XHR should be used.
                //url: the URL being requested
                //protocol: protocol of page requirejs-base64 is running on
                //hostname: hostname of page requirejs-base64 is running on
                //port: port of page requirejs-base64 is running on
                //Use protocol, hostname, and port to compare against the url
                //being requested.
                //Return true or false. true means "use xhr", false means
                //"fetch the .js version of this resource".
            }
        }
    }
});
```

### Custom XHR hooks

There may be cases where you might want to provide the XHR object to use
in the request, or you may just want to add some custom headers to the
XHR object used to make the request. You can use the following hooks:

```javascript
requirejs.config({
    config: {
        text: {
            onXhr: function (xhr, url) {
                //Called after the XHR has been created and after the
                //xhr.open() call, but before the xhr.send() call.
                //Useful time to set headers.
                //xhr: the xhr object
                //url: the url that is being used with the xhr object.
            },
            createXhr: function () {
                //Overrides the creation of the XHR object. Return an XHR
                //object from this function.
            },
            onXhrComplete: function (xhr, url) {
                //Called whenever an XHR has completed its work. Useful
                //if browser-specific xhr cleanup needs to be done.
            }
        }
    }
});
```

### Forcing the environment implementation

The base64 plugin tries to detect what environment it is available for loading
text resources, Node or XMLHttpRequest (XHR), but sometimes the
Node environment may have loaded a library that introduces an XHR
implementation. You can force the environment implementation to use by passing
an "env" module config to the plugin:

```javascript
requirejs.config({
    config: {
        base64: {
            //Valid values are 'node' or 'xhr'
            env: 'rhino'
        }
    }
});
```

## License

Dual-licensed -- new BSD or MIT.

## Where are the tests?

TODO

## History

This plugin is a fork from the [requirejs text plugin](https://raw.github.com/requirejs/text/).
