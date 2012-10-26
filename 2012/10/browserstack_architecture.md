# BrowserStack Support for Testacular

----

So recently I've started working on [Testacular][1] and one of the
features that where requested is support for [BrowserStack][2]. I
wanted to implement it but I had no account with them so I've just
mailed the support and asked if I could have a test account with api
access for creating this feature. (Only paid accounts usually get
access via the api) And sure enough they were nice enough and I had my
free account setup after a few hours!

So now I'm going to document my experience and ideas implementing this
next step for testacular.

## Part 1 - Configuration

### Browsers
My first wish was that the integration should be so seamless that
users could just use it. So I came up with the following idea
``` js
browsers = [
 'Firefox',
 'Chrome',
 // the following are on BrowserStack
 'BS::win::Firefox',
 'BS::win::Firefox::15.0',
 'BS::ios::iPhone4',
 'BS::mac::Safari'
];
```
So you could just add some browser prefixed with `BS::` and let the
tests run. Now of course to use this you need credentials for loging
in. If you don't define anything testacular will look for the
following environment variables
``` bash
BROWSERSTACK_USER = 'user@mail.com'
BROWSERSTACK_PASS = 'secret'
```
but you can set them to other values if you like to.
``` js
browserstack = {
  username: 'user@mail.com',
  password: 'secret'
};
```
As you have seen 


### The tunnel to happiness
For BrowserStack to be able to run your tests you need to provide a
public URL to your testsuite. This is done through a one-time tunnel. 
I didn't lie when I wrote above that you only need to set the browsers
and can run your tests. If there is no tunnel configuration set it
will default to using [localtunnel][3]. But you can use your own
tunnel command. 

Let's say you have an account with [forward][4]. The first thing is
you need to install their gem.
``` bash
$ gem install forward
```
and setup your credentials by just forwarding any test port.
``` bash
$ forward 8000
Already have an account with Forward? y
Enter your email and password
email: user@mail.com
password: ******
Forwarding port 8002 at https://user.fwd.wf
Ctrl-C to stop forwarding
```
Now configure testacular to use it
``` js
tunnel = {
  cmd: 'forward <%= port %>'
  timeout: 5000
};
```
And finished. Testacular will start this tunnel when it needs it and
look for a string starting with `http://` or `https://` to extract the
url for further use. The timeout is the time in ms Testacular will give the
command to return a url before it throws an error.


## Part 2 - Implementation

### The tunnel

For the implementation of the tunnel there two external dependencies
used. The first is [q][5] for creating
promises and second is [lodashs][6] template function for interpolation
the command.

Three options get passed to the tunnel
* `cmd` a template for the command to run
* `port` the port for which a tunnel should be created
* `timeout` in ms (defaults to 5000)

First we create a deferred object with q.
``` js
  var deferred = q.defer();
```
Now we setup everything we need, the command, args and a regexp to
match a url.
``` js
var cmd = _.template(cmd, {port: port});
var args = [];
var timeout = timeout || 5000;
  
var regexp = /(https?:\/\/)?([\da-z\.-]+)\.([a-z\.]{2,6})([\/\w \.-]*)*\/?/;
```
Next the process is spawned.
``` js
  var process = spawn(cmd, args);
```
If the timeout passes reject the promise as this means
we haven't found a valid url in time.
``` js
setTimout(deferred.reject("No valid url found."), timeout);
``` 
All data from the child process output is matched
in search for a valid url
``` js
process.stdout.on('data', function(data){
  var url = data.match(regexp);
  if(url) {
    deferred.resolve(url[0]);
  }
});

process.stderr.on('data', function(data){
  deferred.reject(data);
});
```
And at the end we return a promise.
``` js
  return deferred.promise;
```

[1]: https://github.com/vojtajina/testacular
[2]: http://www.browserstack.com
[3]: https://github.com/shtylman/localtunnel
[4]: https://forwardhq.com/
[5]: https://github.com/kriskowal/q
[6]: http://lodash.com/docs#template
