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

## Part 1 - Architecture

### User Interface or "What goes into the config file?"
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
]
```
So you could just add some browser prefixed with `BS::` and let the
tests run. Now of course to use this you need credentials for loging
in. If you don't define anything testacular will look for the
following environment variables
```bash
BROWSERSTACK_USER = 'username'
BROWSERSTACK_PASS = 'secretpassword'
```



[1]: https://github.com/vojtajina/testacular
[2]: http://www.browserstack.com
