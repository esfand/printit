### Difference between patha and alias ###

There's a subtle difference.

```
# This will result in files being searched for in /foo/bar/bar 
# as the full URI is appended.

location /bar {
    root /foo/bar;
}
```

```
# This will result in files being searched for in /foo/bar 
# as only the URI part after /bar is appended.

location /bar {
    alias /foo/bar;
}
[/code]
```


There is a very important difference between the root and the alias directives. 
This difference exists in the way the path specified in the root or the alias is processed.

In case of the root directive, full path is appended to the root including the location part, whereas 
in case of the alias directive, only the portion of the path NOT including the location part is appended to the alias.

To illustrate...

Let's say we have the config

```
location /static/ {
    root /var/www/app/static/;
    autoindex off;
}
```

In this case the final path that Nginx will derive will be `/var/www/app/static/static`
This is going to return 404 since there is no static/ within static/

This is because the location part is appended to the path specified in the root. 
Hence, with root, the correct way is

```
location /static/ {
    root /var/www/app/;
    autoindex off;
}
```        

On the other hand, with alias, the location part gets dropped. So for the config

```
location /static/ {
    alias /var/www/app/static/;
    autoindex off;
}
```
        
the final path will correctly be formed as `/var/www/app/static`

Following is the documentation of root and alias from http://wiki.nginx.org/HttpCoreModule#alias

### root ###
Syntax:	  root path;   
Default:	root html;   
Context:	http, server, location, if in location

Sets the root directory for requests. For example, with the following configuration

```
location /i/ {
    root /data/w3;
}
```

The /data/w3/i/top.gif file will be sent in response to the “/i/top.gif” request.

The path value can contain variables, except $document_root and $realpath_root.

A path to the file is constructed by merely adding a URI to the value of the root directive. 
If a URI has to be modified, the alias directive should be used.

note: Keep in mind that the root will still append the directory to the request so that a request 
for "/i/top.gif" will not look in "/spool/w3/top.gif" like might happen in an Apache-like alias 
configuration where the location match itself is dropped. Use the alias directive 
to achieve the Apache-like functionality.


### alias ###
Syntax:	alias path;   
Default:	—   
Context:	location   
Defines a replacement for the specified location. For example, with the following configuration

```
location /i/ {
    alias /data/w3/images/;
}
```

on request of “/i/top.gif”, the file /data/w3/images/top.gif will be sent.

The path value can contain variables, except $document_root and $realpath_root.

If alias is used inside a location defined with a regular expression then such
regular expression should contain captures and alias should refer to these captures,
for example:

location ~ ^/users/(.+\.(?:gif|jpe?g|png))$ {
    alias /data/w3/images/$1;
}
```

When location matches the last part of the directive’s value:

```
location /images/ {
    alias /data/w3/images/;
}
```

it is better to use the root directive instead:

```
location /images/ {
    root /data/w3;
}
```

### index Directive ###

The ngx_http_index_module module processes requests ending with the slash character (‘/’).
Such requests can also be processed by the ngx_http_autoindex_module and 
ngx_http_random_index_module modules.

#### Example Configuration ####

location / {
    index index.$geo.html index.html;
}


Syntax:	index file ...;   
Default:	
index index.html;   
Context:	http, server, location

Defines files that will be used as an index. The file name can contain variables. Files are checked in the specified order. The last element of the list can be a file with an absolute path. Example:

```
index index.$geo.html index.0.html /index.html;
```

It should be noted that using an index file causes an internal redirect, and 
the request can be processed in a different location. 
For example, with the following configuration:

```
location = / {
    index index.html;
}

location / {
    ...
}
```

a “/” request will actually be processed in the second location as “/index.html”.


### try-files directive ###

Syntax:	try_files file ... uri;   
try_files file ... =code;   
Default:	—   
Context:	server, location

Checks the existence of files in the specified order and uses the first found file for request processing; the processing is performed in the current context. The path to a file is constructed from the file parameter according to the root and alias directives. It is possible to check directory’s existence by specifying a slash at the end of a name, e.g. “$uri/”. If none of the files were found, an internal redirect to the uri specified in the last parameter is made. For example:

```
location /images/ {
    try_files $uri /images/default.gif;
}

location = /images/default.gif {
    expires 30s;
}
```

The last parameter can also point to a named location, as shown in examples below. Starting from version 0.7.51, the last parameter can also be a code:

```
location / {
    try_files $uri $uri/index.html $uri.html =404;
}
```

Example in proxying Mongrel:

```
location / {
    try_files /system/maintenance.html
              $uri $uri/index.html $uri.html
              @mongrel;
}

location @mongrel {
    proxy_pass http://mongrel;
}
```

Example for Drupal/FastCGI:

```
location / {
    try_files $uri $uri/ @drupal;
}

location ~ \.php$ {
    try_files $uri @drupal;

    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to$fastcgi_script_name;
    fastcgi_param SCRIPT_NAME     $fastcgi_script_name;
    fastcgi_param QUERY_STRING    $args;

    ... other fastcgi_param's
}

location @drupal {
    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to/index.php;
    fastcgi_param SCRIPT_NAME     /index.php;
    fastcgi_param QUERY_STRING    q=$uri&$args;

    ... other fastcgi_param's
}
```

In the following example,

```
location / {
    try_files $uri $uri/ @drupal;
}
```

the try_files directive is equivalent to

```
location / {
    error_page 404 = @drupal;
    log_not_found off;
}
```

And here,

```
location ~ \.php$ {
    try_files $uri @drupal;

    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to$fastcgi_script_name;

    ...
}
```

try_files checks the existence of the PHP file before passing the request to the FastCGI server.


Example for Wordpress and Joomla:

```
location / {
    try_files $uri $uri/ @wordpress;
}

location ~ \.php$ {
    try_files $uri @wordpress;

    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to$fastcgi_script_name;
    ... other fastcgi_param's
}

location @wordpress {
    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to/index.php;
    ... other fastcgi_param's
}
```
