<div class="entry">
		
		<h2><a href="http://blog.rackcorp.com/2010/05/nginx-location-and-rewrite-configuration-made-easy/" rel="bookmark" title="Permanent Link to Nginx location and rewrite configuration made easy">Nginx location and rewrite configuration made easy</a></h2>


<p>Okay guys, so as many of you know, we offer both Apache and Nginx servers here as part of our standard shared hosting packages.  There is no better web server out there for reliable performance in a high-traffic environment. One thing that I frequently go through with the new staff here are nginx location / rewrite rules because they can be a bit confusing.</p>

<p>The best way to think of things is that as a request comes in, Nginx will scan through the configuration to find a “location” line that matches the request.  There are TWO modes that nginx uses to scan through the configuration file: literal string matching and regular expression checks.  Nginx first scans through ALL literal string location entries in the order that they occur in the configuration file, and secondly scans through ALL the regular expression location entries in the order that they occur in the configuration file.  So be aware – location ordering order DOES matter.</p>

<p>Now there’s a few ways of interrupting that flow:</p>

<blockquote><p><code>location = /images { }</code> (Note: does not work for regular expressions)<br>
The “=” is the important character here.  This matches a request for “/images” ONLY.  This also halts the location scanning as soon as such an exact match is met.</p>
<p><code>location ^~ /images {}</code> (Note: does not work for regular expressions)<br>
The “^~” results in a case sensitive match for the beginning of a request. This means /images, /images/logo.gif, etc will all be matched.  This also halts the location scanning as soon as a match is met.</p>

<p><code>location ~ /images {}</code><br>

<code>location ~* /images {}</code> (case insensitive version)<br>

This causes a case (in-)sensitive match for the beginning of a request.  Identical to the previous one, except this one doesn’t stop searching for a more exact location clauses.</p></blockquote>

<p>That’s IT!  Yes it really is that simple.  Now there’s a few variations for case-insensitive matches or named-locations, but don’t worry about those for now. </p>

<p>Now all of the above examples are literal string examples.  If you replace /images with a regular expression then suddenly you have altered the order of the rules (remember ALL literal strings get checked first, and THEN regular expressions – regardless of the order you have them in your configuration).</p>

<p>An examples of a regular expression match is:</p>

<blockquote><p><code>location ~ \.(gif|jpg|jpeg)$ { }</code><br>

This will match any request that ends in .gif, .jpg, or .jpeg.</p></blockquote>

<p>So now that we’ve discussed the foundations of the location rules, we can move into rewrites.  There are TWO kinds of rewrites – URL redirects (HTTP301/HTTP302), or an internal rewrite (mangles the request before it is processed).</p>

<p>URL Redirects are the simplest to understand:</p>

<blockquote>
<p>
location /admin {<br>
rewrite ^/admin/(.*)$ http://admin.example.com/$1 permanent;<br>
}
</p>
</blockquote>

<p>This example will redirect any request matching the location rule (see earlier) as a HTTP 301 permanent redirection to http://admin.example.com/.  e.g. http://www.example.com/admin/index.html now gets HTTP redirected to http://admin.example.com/index.html.  Note the regular expression and the $1 replacement in the URL.  If you want the redirect to be a HTTP 302 (temporary redirection), just change the word “permanent” to “redirect”.</p>

<p>Internal rewrites are a little more complicated:</p>

<blockquote>
<p>
location /admin {<br>
rewrite ^/admin/(.*)$ /$1 break;<br>
}
</p>
</blockquote>

<p>The key word here is “break”.  This causes the rewrite processing to stop.  If this word was “last”, it would then go back to scanning location entries as per our discussions earlier – but now with the rewritten URL.</p>

<p>
I hope that clears up nginx configuration.  The documentation is really good over at the nginx wiki (http://wiki.nginx.org/NginxModules).  I think this was the only part that sometimes confuses some of us here.  let us know if you think I missed anything, otherwise I hope to put up some of our nginx rewrites for some o the more popular forums/blogs in the weeks to come!
</p>

<div class="bookmarkify">
<a name="bookmarkify"></a>
<div class="linkbuttons">
<a href="http://del.icio.us/post?url=http://blog.rackcorp.com/2010/05/nginx-location-and-rewrite-configuration-made-easy/&amp;title=Nginx location and rewrite configuration made easy" title="Save to del.icio.us" onclick="target=&quot;_blank&quot;;" rel="nofollow">
<img src="http://zczqa7hw9.fastestcdn.net/wp-content/plugins/bookmarkify/delicious.ico" style="width:16px; height:16px;" alt="[del.icio.us]">
</a> 
<a href="http://digg.com/submit?phase=2&amp;url=http://blog.rackcorp.com/2010/05/nginx-location-and-rewrite-configuration-made-easy/&amp;title=Nginx location and rewrite configuration made easy" title="Digg It!" onclick="target=&quot;_blank&quot;;" rel="nofollow">
<img src="http://zczqa7hw9.fastestcdn.net/wp-content/plugins/bookmarkify/digg.ico" style="width:16px; height:16px;" alt="[Digg]">
</a> 
<a href="http://www.stumbleupon.com/submit?url=http://blog.rackcorp.com/2010/05/nginx-location-and-rewrite-configuration-made-easy/&amp;title=Nginx location and rewrite configuration made easy" title="Stumble It!" onclick="target=&quot;_blank&quot;;" rel="nofollow">
<img src="http://zczqa7hw9.fastestcdn.net/wp-content/plugins/bookmarkify/stumbleupon.ico" style="width:16px; height:16px;" alt="[StumbleUpon]">
</a> 
<a href="http://technorati.com/faves?add=http://blog.rackcorp.com/2010/05/nginx-location-and-rewrite-configuration-made-easy/" title="Add to my Technorati Favorites" onclick="target=&quot;_blank&quot;;" rel="nofollow">
<img src="http://zczqa7hw9.fastestcdn.net/wp-content/plugins/bookmarkify/technorati.ico" style="width:16px; height:16px;" alt="[Technorati]">
</a> 
<a href="https://favorites.live.com/quickadd.aspx?mkt=en-us&amp;url=http://blog.rackcorp.com/2010/05/nginx-location-and-rewrite-configuration-made-easy/&amp;title=Nginx location and rewrite configuration made easy" title="Save to Windows Live" onclick="target=&quot;_blank&quot;;" rel="nofollow">
<img src="http://zczqa7hw9.fastestcdn.net/wp-content/plugins/bookmarkify/windowslive.ico" style="width:16px; height:16px;" alt="[Windows Live]">
</a> 
</div>
</div>		
</div>
