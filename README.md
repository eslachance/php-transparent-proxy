# PHP Transparent Proxy Script

Proxy (noun, /'prɒksi/): A process that accepts requests for some service and passes them on to the real server. ([source](http://foldoc.org/proxy))

I needed to do cross-domain AJAX calls from a jQuery front-end to a PHP backend which was on another domain, and couldn't find a complete, functional proxy script that could bridge both ends... so I created my own. Since both servers had PHP (but the backend needed extra stuff that wasn't on the frontend server), doing a PHP Proxy seemed like the way to go.

## No Longer Maintained

Please note that I no longer maintain this repo, and have no plans to update it in the future.

### What this is *NOT*

PHP Transparent Proxy is not a server proxy that redirects requests. In other words, it's not a server that you would add to your browser and that would "pretend" to be from somewhere else. It can't be used to bypass a firewall at work or at school, etc.

### No Extended Email Support

While I will attempt to guide someone or answer email requests sent to me, I am not in a position to offer extended support, debugging, or remote support to implement the use of this proxy or debug your code. In other words, this code is provided as-is!

### What this proxy supports:

- GET and POST requests (POST was the whole reason for this, since jsonp doesn't support it!)
- HTTP_REFERER check (only accept requests from one server)
- COOKIES, in both directions. Technically part of HEADERS, but it's worth mentioning!
- HEADERS in both directions (with specific exceptions, see Limitations).

### What it doesn't support (yet, maybe):

- Dynamic destination (though that's relatively trivial to change), because I don't need it.
- Load Balancing/Cycling, I may add this as a personal exercise in the future.
- Authentication, beyond the referrer check, or session (this should be handled by the backend anyway)
 
### Quick Steps

- Download proxy.php. 
- Modify proxy.php and change the $destinationURL to the URL where your backend server-side script is (meaning, where you want to pull the data from)
- Place the proxy.php script somewhere on your website hosting, on the same server you want to use it from.
- Change your front-end JavaScript to use the proxy.php instead of the backend URL. See POST Example section below for an example usage.

## Usage Examples

This page provides some example for using this proxy from different locations. Please do not hesitate to let me know if you've successfully used the proxy by some other method as well as a code example for it.

Really though, I do this only to have a wiki page. If you need this proxy, you probably know how to make an HTTP Request and you don't need me to show you!

### Calling from jQuery with .ajax()

The main reason for the existence of this proxy is to use it for cross-domain HTTP POST ajax calls using jQuery, so this is currently the only example I have for you. This code assumes you've already loaded jquery on your page (d'uh!).

#### POST Example

The example itself loads from a PHP backend and checks whether the user is logged on (uses the PHPSESSID cookie automatically, which explains cookie support!), and sets some local javascript variables.

    $.ajaxSetup({
     url: '/path/to/proxy.php',
     contentType:"application/x-www-form-urlencoded",
     type:"POST",
     cache:false,
     dataType:"json",
    });

    $.ajax({
     data:{ action:'checkLogon' },
     success:function(data) {
      if(data.response[0].answer === 'true') {
       loggedin = true;
       isAdmin = data.response[0].isAdmin;
       if(isAdmin) {
        loadJS("admintools.js");
       }
       fullName = data.response[0].userFullName;
      }
     }      
    });

#### GET Example

This example loads some global parameters from my backend (such as translated strings and menu items from a database). As you can see, any ajax query or function should work fine.

    $.getJSON("/common/proxy.php",{action: 'getglobals', suite:whichSuite, soft:whichGuide}, function(data) {
     $("#versionLabel").html(data.globals.versionLabel[0][currentLang]);
     $("#lastUpdateLabel").html(data.globals.lastUpdLabel[0][currentLang]);
     var ip = data.globals.visitorInfo[0]['ip'];
     $.each(data.globals.docs, function(i,item) {
      // here I add each item to a menu UL.
     }
    });

## Limitations

### Headers

While this proxy attempts to be as transparent as possible, there are a couple of things that break this (unless I can find a way to fix them in the future).

#### Headers from the backend to the client

Any and all headers should be sent from the backend to the client with no modification. Note however that PHP simply overwrites existing headers with the header() function. If the PHP proxy or the server on which it resides adds headers that are not existent on the backend, they will remain.

So, you have the headers from the backend untouched along with any extra headers on the proxy that aren't overwritten.

#### Headers from the client to the backend

The following headers are forced by the proxy:

* "Host: " is forced because, obviously, it's the address of the backend we need to set.
* "GET: " or "POST: " is forced because we are addressing a different path as well as adding the query string for GET requests.
* "Accept-Charset: " is forced to `ISO-8859-1,utf-8;q=0.7,*;q=0.7` because it doesn't work without it, beats me why (I get a 400 error)
* "Connection: " is forced to `close` at the end of the request (before the data) because keep-alive isn't supported.

That last bit means that this proxy is non-streaming. If it were, my browser sending "Connection: keep-alive" would work fine and I'd jump for joy... But that's not the case.
