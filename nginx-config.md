
### Nginx location and rewrite configuration made easy ###

As a request comes in, Nginx will scan
through the configuration to find a `location` line that matches the request.
There are TWO modes that nginx uses to scan through the configuration file:
**literal string matching** and **regular expression checks**.
Nginx first scans through ALL **literal string location entries** in the order that
they occur in the configuration file, and 
secondly scans through ALL the **regular expression location entries** in the order that 
they occur in the configuration file.
So be aware – location ordering order DOES matter.

Now there’s a few ways of interrupting that flow:

```
location = /images { }   (Note: does not work for regular expressions)
```

The **=** is the important character here.  This matches a request for `/images` ONLY.
This also halts the location scanning as soon as such an exact match is met.

```
location ^~ /images {}   (Note: does not work for regular expressions)
```
    
The **^~** results in a case sensitive match for the beginning of a request.
This means `/images`, `/images/logo.gif`, etc will all be matched.
This also halts the location scanning as soon as a match is met.

```
location ~ /images {}
location ~* /images {}  (case insensitive version)
```

This causes a case (in-)sensitive match for the beginning of a request.
Identical to the previous one, except this one doesn’t stop searching
for a more exact location clauses.

That’s IT!  Yes it really is that simple.
Now there’s a few variations for case-insensitive matches or named-locations,
but don’t worry about those for now.

Now all of the above examples are literal string examples.
If you replace `/images` with a regular expression then suddenly you have altered the order of the rules
(remember ALL **literal strings** get checked first, and THEN **regular expressions** –
regardless of the order you have them in your configuration).

An examples of a regular expression match is:

    location ~ \.(gif|jpg|jpeg)$ { }

This will match any request that ends in `.gif`, `.jpg`, or `.jpeg`.

So now that we’ve discussed the foundations of the location rules, we can move into rewrites.
There are TWO kinds of rewrites – **URL redirects** (HTTP301/HTTP302), or an **internal rewrite**
(mangles the request before it is processed).

URL Redirects are the simplest to understand:

```
location /admin {<br>
    rewrite ^/admin/(.*)$ http://admin.example.com/$1 permanent;
}
```

This example will redirect any request matching the location rule (see earlier)
as a HTTP 301 permanent redirection to `http://admin.example.com/`.
e.g. `http://www.example.com/admin/index.html` now gets HTTP redirected 
to `http://admin.example.com/index.html`.
Note the regular expression and the **$1** replacement in the URL.
If you want the redirect to be a HTTP 302 (temporary redirection),
just change the word **permanent** to **redirect**.

Internal rewrites are a little more complicated:

```
location /admin {<br>
    rewrite ^/admin/(.*)$ /$1 break;
}
```

The key word here is **break**.  This causes the rewrite processing to stop.
If this word was **last**, it would then go back to scanning location entries
as per our discussions earlier – but now with the rewritten URL.
