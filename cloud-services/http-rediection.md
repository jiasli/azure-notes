# Azure Cloud Serivce HTTPS Redirection and Canonical Host Name Redirection

IIS's built-in **HTTP Redirect** module is very simple and can only do an unconditional redirection.

For a more complicated URL redirection, we need to use IIS's **URL Rewrite** module. On an **Azure Virtual Machine** or local Windows Service machine, we need to manually install **URL Rewrite** module from https://www.iis.net/downloads/microsoft/url-rewrite. **Azure Cloud Serivce** already has this module installed, so [startup task](https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-startup-tasks) is not needed.

Starting from https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/url-rewrite-module-configuration-reference, which contains an official sample to redirect to canonical URL.


```xml
<rule name="Redirect to canonical url">
  <match url="^(.+)" />
  <!-- rule back-reference is captured here -->
  <conditions>
    <!-- Check whether the requested domain is in canonical form -->
    <add input="{HTTP_HOST}" type="Pattern" pattern="^www\.mysite\.com$" negate="true" />
  </conditions>
  <!-- Redirect to canonical url and convert URL path to lowercase -->
  <action type="Redirect" url="http://www.mysite.com/{tolower:{R:1}}" RedirectType="Found"/>
</rule>
```

We change it to incoperate HTTPS redirection as well.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        <rule name="redirectazure" stopProcessing="true">
          <match url="(.*)" />
          <conditions logicalGrouping="MatchAny">
            <add input="{HTTP_HOST}" pattern="^www\.mysite\.com$" negate="true" />
            <add input="{SERVER_PORT_SECURE}" pattern="0" />
          </conditions>
          <action type="Redirect" url="https://www.mysite.com/{tolower:{R:1}}" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```

This can also be configured in **Internet Information Services (IIS) Manager** with GUI. Then copy the contents from local **web.config** to **web.config** in the Cloud Service project.
