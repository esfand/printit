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

