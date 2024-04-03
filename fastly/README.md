<!--
Copyright (c) 2024 The C++ Alliance, Inc. (https://cppalliance.org)

Distributed under the Boost Software License, Version 1.0. (See accompanying
file LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)

Official repository: https://github.com/boostorg/website-v2
-->

## Fastly Notes

A Fastly CDN is configured as a front-end to the stage and production sites.

Configuration Steps:

- Create a Delivery Service

- Set the domain. www.boost.org

- Add a Host. Use the IP address of the backend, such as 35.244.221.26 : 443 . Disable SSL checks if that hasn't been set up yet.  

- Override host header: www.boost.org

- Force TLS and enable HSTS. Enabled this setting after certificates have been installed.  

- Redirect traffic to www subdomains is not needed. A visitor to the basic domain (boost.org) will hit GCP directly, and be redirected to www. This is due to the fact that Fastly certificates conflict with Google certificates. The way to solve the problem currently is to provision the boost.org & *.boost.org on GCP, while using www.boost.org on Fastly. In this way, the domains do not exactly overlap, and certificates can be created on both hosting providers.

- VCL Snippets:  

'Remove certain cookies'

If more cookies are added to the website, they must be included in this snippet.

```
# Some generic cookie manipulation, useful for all templates that follow
# Remove the "has_js" cookie
set req.http.Cookie = regsuball(req.http.Cookie, "has_js=[^;]+(; )?", "");

# Remove any Google Analytics based cookies
set req.http.Cookie = regsuball(req.http.Cookie, "__utm[^=]+=[^;]+(; )?", "");
set req.http.Cookie = regsuball(req.http.Cookie, "_ga[^=]*=[^;]+(; )?", "");
set req.http.Cookie = regsuball(req.http.Cookie, "_gcl_[^=]+=[^;]+(; )?", "");
set req.http.Cookie = regsuball(req.http.Cookie, "_gid=[^;]+(; )?", "");

# Remove DoubleClick offensive cookies
set req.http.Cookie = regsuball(req.http.Cookie, "__gads=[^;]+(; )?", "");

# Remove the Quant Capital cookies (added by some plugin, all __qca)
set req.http.Cookie = regsuball(req.http.Cookie, "__qc.=[^;]+(; )?", "");

# Remove the AddThis cookies
set req.http.Cookie = regsuball(req.http.Cookie, "__atuv.=[^;]+(; )?", "");

if ( req.url ~ "^/doc/" ) {
  # unset req.http.Cookie:csrftoken;
  # unset req.http.Cookie:config-sessionid;
  # Let's remove all cookies in /doc/ :
  unset req.http.Cookie;
}

# Remove a ";" prefix in the cookie if present
set req.http.Cookie = regsuball(req.http.Cookie, "^;\s*", "");

# Are there cookies left with only spaces or that are empty?
if (req.http.cookie ~ "^\s*$") {
  unset req.http.cookie;
}
``` 

'Doc cache age'

```
if ( req.url ~ "^/doc/" ) {
  set beresp.ttl = 604800s;
  return (deliver);
}
```
