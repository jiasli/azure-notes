# Azure CDN Common Verizon Premium Rules Engine

## Attach index.html to Root

For example we have bought a domain www.myweb.com and pointed CNAME of this domain to **Azure Verizon Premium CDN** endpoint myweb.azureedge.net. We want to visit www.myweb.com without /index.html, Rules Engine is our best choice.

In this way, we can host our website on **Azure Blob Storage** and use **Azure CDN** as the frontend.

In **Manage** Portal, add a rule as following:

```xml
<match.always>
  <feature.url-user-rewrite pattern="/8071111/myweb/$" value="/807117F/myweb/index.html" />
</match.always>
```

Please note that internally the starting `^` of the RegEx is actually before the `pattern`. So the internal pattern is `^/8071111/myweb/$`. Don't place `^` in the textbox as this will internally become `/8071111/myweb/^$`.

## HTTPS redirection

```xml
<match.request-scheme value="http">
  <feature.url-redirect code="301" pattern="/8075FFC/endpointname/(.*)" value="https://endpointname.azureedge.net/$1" />
</match.request-scheme>
```
