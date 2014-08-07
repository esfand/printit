

# Serving Static Content #

This article describes how to serve static content, how to use different ways of 
setting up the paths to look for files, and how to set up index files.

## Root Directory and Index Files ##

The root directive specifies the root directory which will be used to search for a file. 
To obtain the path of a requested file, NGINX adds the request URI added to the path 
specified in *root*.

The directive can be placed on any level within the **http**, **server**, or **location** contexts. 
In the example below, the <em>root</em> directive is defined for a virtual server. 
It will be applied to all locations where the root is not redefined:

```
server {
    root /www/data;

    location / {
    }

    location /images/ {
    }

    location ~ \.(mp3|mp4) {
        root /www/media;
    }
}
```

Here, the `/images/some/path` URI will be mapped to `/www/data/images/some/path`
on the file system, and NGINX will try to get a file there.
A request with a URI such as `/any/path/file.mp3` will be mapped to `/www/media/any/path/file.mp3` 
because the corresponding location defines its own root.

If a request ends with a slash, NGINX will treat it as a request for a directory and
will try to find an index file there. The name of the index file is specified in the
`index` directive, the default value is index.html. In the example above, to the request
with the URI `/images/some/path/` NGINX will respond with `/www/data/images/some/path/index.html`
if that file exists. If this file does not exist, a 404 error will be returned by default.
It is possible, however, to return an automatically generated directory listing when the
index file does not exist by setting the autoindex directive to *on*.

```
location /images/ {
    autoindex on;
}
```

<p>The <em>index</em> directive can list more than one file name. Each file will be checked in the order listed, and the first file that exists will be returned.</p>


```
location / {
    index index.$geo.html index.htm index.html;
}
```

<p>The <em>$geo</em> variable here is a custom variable set through the <a href="http://nginx.org/en/docs/http/ngx_http_geo_module.html#geo">geo</a> directive. The value of the variable depends on the client’s IP address.</p>

<p>To return the index file, NGINX checks its existence and then makes an internal redirect to the URI obtained from the index file name and the base URI. The internal redirect results in a new search of a location and can end up in another location as in the following example:</p>

```
location / {
    root /data;
    index index.html index.php;
}

location ~ \.php {
    fastcgi_pass localhost:8000;
    ...
}
```

<p>Here, if a request has the <code>/path/</code>URI, and it turns out that <code>/data/path/index.html</code> does not exist, but <code>/data/path/index.php</code> does, the internal redirect to <code>/path/index.php</code> will be mapped to the second location. As a result, the request will be proxied.</p>

## Trying Several Options ##

<p>The <a href="http://nginx.org/en/docs/http/ngx_http_core_module.html#try_files">try_files</a> directive can be used to check whether the specified file or directory exists and make an internal redirect, or return a specific status code if they don’t. For example, to check the existence of a file corresponding to the request URI, use the <em>try_files</em> directive and the $uri variable as follows:</p>

```
server {
    root /www/data;

    location /images/ {
        try_files $uri /images/default.gif;
    }
}
```

<p>The file is specified in the form of the URI, which is processed using the <em>root</em> or <em>alias</em> directives set in the context of the current location or virtual server. In this case, if the file corresponding to the original URI doesn’t exist NGINX makes an internal redirect to the URI specified in the last parameter returning <code>/www/data/images/default.gif</code>.</p>

<p>The last parameter can also be a status code (specified after =) or the name of a location. In the following example, a 404 error is returned if none of the options resolves into an existing file or directory.</p>

```
location / {
    try_files $uri $uri/ $uri.html =404;
}
```

<p>In the next example if neither the original URI, nor the URI with the appended trailing slash, resolve into an existing file or directory, the request is redirected to the named location which passes it to a proxied server.</p>

```
location / {
    try_files $uri $uri/ @backend;
}

location @backend {
    proxy_pass http://backend.example.com;
}
```
