## Location Selection Algorithm ##

Let’s look at how nginx chooses a location to process a request for a typical, 
simple PHP site:

```
server {
    listen      80;
    server_name example.org www.example.org;
    root        /data/www;

    location / {
        index   index.html index.php;
    }

    location ~* \.(gif|jpg|png)$ {
        expires 30d;
    }

    location ~ \.php$ {
        fastcgi_pass  localhost:9000;
        fastcgi_param SCRIPT_FILENAME
                      $document_root$fastcgi_script_name;
        include       fastcgi_params;
    }
}
```

nginx first searches for the *most specific prefix location* given by **literal strings** 
regardless of the listed order.  In the configuration above the only **prefix location**
is “/” and since it matches any request it will be used as a last resort. 
Then nginx checks locations given by regular expression in the order listed in the configuration file. 
The first matching expression stops the search and nginx will use this location. 
If no regular expression matches a request, then nginx uses the most specific 
prefix location found earlier.

Note that locations of all types test only a URI part of request line without arguments.
This is done because arguments in the query string may be given in several ways,
for example:

```
/index.php?user=john&page=1
/index.php?page=1&user=john
```

Besides, anyone may request anything in the query string:

```
/index.php?page=1&something+else&user=john
```

Now let’s look at how requests would be processed in the configuration above:

* A request **/logo.gif** is matched by the prefix location `/` first and then 
  by the regular expression `\.(gif|jpg|png)$`, therefore, it is handled by the latter location. 
  Using the directive `root /data/www` the request is mapped to the file **/data/www/logo.gif**, 
  and the file is sent to the client.

* A request **/index.php** is also matched by the prefix location `/` first and 
  then by the regular expression `\.(php)$`. Therefore, it is handled by the latter location and 
  the request is passed to a FastCGI server listening on localhost:9000. 
  The fastcgi\_param directive sets the FastCGI parameter SCRIPT\_FILENAME 
  to **/data/www/index.php**, and the FastCGI server executes the file. 
  The variable $document\_root is equal to the value of the root directive and the 
  variable $fastcgi\_script\_name is equal to the request URI, i.e. “/index.php”.

* A request **/about.html** is matched by the prefix location “/” only, therefore, 
  it is handled in this location. Using the directive `root /data/www` the request 
  is mapped to the file **/data/www/about.html**, and the file is sent to the client.

* Handling a request **/** is more complex.
  It is matched by the prefix location “/” only, therefore, it is handled by this location. 
  Then the index directive tests for the existence of index files according to its parameters and 
  the `root /data/www` directive. If the file /data/www/index.html does not exist, and the 
  file /data/www/index.php exists, then the directive does an **internal redirect** to 
  “/index.php”, and nginx searches the locations again as if the request had been sent by a client. 
  As we saw before, the **redirected request** will eventually be handled by the FastCGI server.
