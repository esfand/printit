The Rule of nginx location Match
================================
[Source](https://github.com/seansoong/songjinshan.com_blog/blob/master/articles/linux/the-rule-of-nginx-location-match.md)

`location` might be the most fundamental config directive of nginx, but the rule about which url 
should match which `location` config section has always been a mystery.  In fact, the rule is much 
less complicated than the online documentation says about it.

## Syntax

I state the rule in my own way.

Syntax: `location [ = | ~ | ~* | ^~ ] uri { ... }`

Between **location** and **uri** there can be one of these four **operators** `=`, `~`, `~*`, `^~`, or there can 
be `no operator` at all. For brevity I'll say the operator is `none` in the case where there is no 
operator at all.

NOTE The *operator*, as I call it, is actually called *prefix* in nginx documentation. I intentionally 
avoid *prefix* and use *operator* instead, to make it sound more natural and less confusing.

If the operator is **=**, **^~** or **none**, the **uri** part in the directive is a string literal and is matched 
against request uri or the beginning part of request uri (i.e. its prefix). Operator **=** indicates an 
exact match, which means the `uri` part in the directive should be exactly the same as request uri. 
Operator ^~ or none indicates a prefix match, which means the `uri` part in the directive can be the 
whole request uri or only the beginning part of it. 

NOTE When referring to request uri, I only talk about the part after hostname and before 
query string (looks like an absolute file path).

If the operator is ~ or ~\*, the `uri` part in the directive is a regular expression, 
with operator ~ for case sensitive match and operator ~\* for case insensitive match.

## The Matching Rule

The actual matching rule goes as follows.

Considering all string literal `uri`s, find the longest matching one against request uri:

1. If the longest matching one has the operator = or ^~, the whole matching phase is finished
   and that one is the final result (which means nginx will get on with its directives in `{ ... }`).

2. If the longest matching one has the operator none, nginx will continue to test all 
   regular expression `uri`s one by one (following the order they appear in the config file)

   2a. If nginx find one regular expression `uri` that matches request uri,
       the whole matching phase is finished and this regular expression `uri` is the final result.

   2b. If no regular expression `uri` matches request uri, the whole matching phase is finished and
       the original longest matching string literal `uri` is the final result.

3. If no string literal `uri` matches request uri, nginx will continue to test all regular 
   expression `uri`s one by one (following the order they appear in the config file)

   3a. If nginx find one regular expression `uri` that matches request uri, the whole matching
       phase is finished and this regular expression `uri` is the final result.

   3b. If no regular expression `uri` matches request uri, we conclude that not a
       single `location` directive matches request uri and 404 Not Found should be returned.

## Test Case

A test config file like the following might be helpful to understand the matching rule:

```text
location = /a {
    return 500;
}
location ^~ /a/b {
    return 501;
}
location /a/b/c {
    return 502;
}
location ~ b {
    return 503;
}
location ~* c {
    return 504;
}
```
