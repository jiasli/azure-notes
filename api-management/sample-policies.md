# Azure API Management Sample Policies
This documents includes some sample Azure API Management Policies that is useful for daily use.

API Management advanced policies: https://docs.microsoft.com/en-us/azure/api-management/api-management-advanced-policies

## HTTPS Redirection

You may use <return-response> Policies to achieve to HTTPS redirection. We detect if the OriginalURL contains http://. If so, return 301 and attach Location header in the response, replacing http:// with https://.

```xml
<policies>
    <inbound>
        <choose>
            <when condition="@(context.Request.OriginalUrl.ToString().Contains("http://"))">
                <return-response>
                    <set-status code="301" />
                    <set-header name="Location" exists-action="override">
                        <value>@(context.Request.OriginalUrl.ToString().Replace("http://","https://"))</value>
                    </set-header>
                </return-response>
            </when>
        </choose>
    </inbound>
    <backend>
        <forward-request />
    </backend>
    <outbound />
    <on-error />
</policies>
``` 


Testing result:


```
>curl "http://jlapim.azure-api.net/text/probe.txt" -L -v
 
> GET /text/probe.txt HTTP/1.1
> Host: jlapim.azure-api.net
> User-Agent: curl/7.53.1
> Accept: */*
> 
< HTTP/1.1 301 Moved Permanently
< Content-Length: 0
< Location: https://jlapim.azure-api.net/text/probe.txt
< Date: Tue, 30 Jan 2018 07:16:51 GMT
 
* Issue another request to this URL: 'https://jlapim.azure-api.net/text/probe.txt'
 
> GET /text/probe.txt HTTP/1.1
> Host: jlapim.azure-api.net
> User-Agent: curl/7.53.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Content-Length: 4720
< Content-Type: text/plain
< Content-MD5: RXYKrlcUXL+nZf5aYH58Rw==
< Last-Modified: Fri, 22 Dec 2017 07:46:53 GMT
< ETag: 0x8D549102B3D3755
```

## gzip Compression

Adding `Content-Encoding: gzip` header on the response will force API Management to compress the response.

The following policy detects the presence of `Accept-Encoding: gzip` and compress the response accordingly. 

```xml
<policies>
    <inbound>
        <set-variable name="gzipresponse" value="@(context.Request.Headers.GetValueOrDefault("Accept-Encoding", "null").Contains("gzip"))" />
    </inbound>
    <backend>
        <forward-request />
    </backend>
    <outbound>
        <choose>
            <when condition="@(context.Variables.GetValueOrDefault<bool>("gzipresponse"))">
                <set-header name="Content-Encoding" exists-action="override">
                    <value>gzip</value>
                </set-header>
            </when>
        </choose>
    </outbound>
</policies>
```

Set HTTP header: https://docs.microsoft.com/en-us/azure/api-management/api-management-transformation-policies#SetHTTPheader

Control flow: https://docs.microsoft.com/en-us/azure/api-management/api-management-advanced-policies#choose
