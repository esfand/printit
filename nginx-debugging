## Nginx Debugging ##

NGinx do not provide so much help (by default) when it comes to debugging internal redirect, 
proxying and other rewrite rules. But it comes with a handy debug module which allows you 
to get a lot more info.

You have to enable it at compile time:

```
./configure --with-debug
```

And then in your configuration you can set:

```
http {
  # At the http level activate debug log for eveything
  error_log /path/to/my/detailed_error_log debug;

  server {
    # At the server level activate debug log only for this server
    error_log /path/to/my/detailed_error_log debug;
  }

  server {
    # At the server level without the debug keywords it disable debug for this server
    error_log /path/to/my/error_log;
  }
}
```

You can even debug only some connections:

```
error_log /path/to/my/detailed_error_log debug;

events {
    debug_connection   10.0.0.1;
    debug_connection   10.0.1.0/24;
}
```


## Enabling debugging log ##

To enable a debugging log, nginx needs to be configured to support debugging during the build:

```
./configure --with-debug ...
```

Then the debug level should be set with the error_log directive:

```
error_log /path/to/log debug;
```

The nginx binary version for Windows is always built with the debugging log support, 
so only setting the debug level will suffice.

Note that redefining the log without also specifying the debug level will disable the debugging log. 
In the example below, redefining the log on the server level disables the debugging log for this server:

```
error_log /path/to/log debug;

http {
    server {
        error_log /path/to/log;
        ...
```

To avoid this, either the line redefining the log should be commented out, or 
the debug level specification should also be added:

```
error_log /path/to/log debug;

http {
    server {
        error_log /path/to/log debug;
        ...
```

It is also possible to enable the debugging log for selected client addresses only:

```
error_log /path/to/log;

events {
    debug_connection 192.168.1.1;
    debug_connection 192.168.10.0/24;
}
```

