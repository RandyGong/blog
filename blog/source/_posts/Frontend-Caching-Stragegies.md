---
title: Frontend Caching Stragegies
date: 2022-01-24 21:12:47
tags:
- cache
- frontend
---

## Frontend cache

Front end caching is an old topic and is often used as a knowledge point in front-end interview. Let's sum up today.

## Category

The front-end cache is divided into strong cache and negotiation cache.

### Strong cache

Strong caching mainly uses Expires and Cache-Control header fields. Both exist at the same time, and cache control has higher priority. When the strong cache is hit, the client will no longer ask, directly read the content from the cache and return the HTTP status code 200.

### Expires

> Response header representing the expiration time of the resource. Is a standard time in GMT format.

When the client requests the server, the server will return the resource and bring the response header expires, which indicates the specific expiration time of the resource. If the client obtains the resource again before the expiration time, it does not need to request my server any more, and you can get it directly in the cache.

Advantages of using expires strong caching

 - Within the expiration time, it saves a lot of traffic for users.
 - Reduces the pressure on the server to repeatedly read disk files.

Disadvantages of using expires strong cache

 - After the cache expires, the server will request the server again regardless of whether the file has changed.
 - The cache expiration time is a specific time, which depends on the time of the client. If the time is inaccurate or changed, the cache will also be affected.

### Cache-Control

> Request / response header, cache control field, and precisely control the cache policy.

To make strong caching more accurate, HTTP 1.1 added the Cache-Control field. Cache-Control can appear in both the request header and the response header. Its different values represent different meanings. Let's analyze them in detail below.

### Cache-Control server side parameters

- max-age: The number of seconds valid is a relative time, which is more accurate than the specific time of expires
- s-maxage: It is used to indicate the effective time of the cache on the cache server (such as cache CDN, cache proxy server), and is only valid for public cache
- no-cache: Do not use local strong caching. Cache negotiation is required
- no-store: Directly prohibit the browser from caching data. Every time the user requests the resource, he will send a request to the server and download the complete resource every time
- public: It can be cached by all users, including end users and intermediate proxy servers
- private: default value, it can only be cached by the end user's browser. Intermediate caching agents are not allowed to cache

### Cache-Control client side parameters

- max-stale: 5 means that when the client gets the cache from the proxy server, it doesn't matter even if the proxy cache expires. As long as the expiration time is within 5 seconds, it can still be obtained from the proxy
- minx-fresh: 5 means that the proxy cache needs a certain freshness. Don't wait until the cache just expires. Be sure to take it 5 seconds before the expiration, otherwise you can't get it
- only-if-cached: When this field is added, it means that the client will only accept the proxy cache and will not accept the response from the source server. If the proxy cache is invalid, it directly returns 504 (Gateway timeout)

## Negotiation cache
The negotiation cache mainly has four header fields, which are used in combination, If-Modified-Since and Last-Modified, Etag and If-None-Match. When they exist at the same time, Etag and If-None-Match will prevail. When the negotiation cache is hit, the server will return the HTTP status code 304 to let the client directly read the file from the local cache.

- If-Modified-Since
> The request header, the latest modification time of the resource, is told by the browser to the server. In fact, it is the last modified value returned by the server for the first time

- Last-Modified
> The response header, the latest modification time of the resource, and the server tells the browser

- Etag
> Response header, resource ID, which is told by the server to the browser

- If-None-Match
> Request header, cache resource ID, and the browser tells the server. In fact, it is the Etag value returned by the server for the first time

### If-Modified-Since & Last-Modified

When the client requests the server for the first time, the server will return a last modified response header, which is a standard time. When the client requests the server, it will bring the if modified since request header field. The value of this field is the last modified value returned by the server. After receiving the request, the server will compare whether the two values are the same. If they are the same, it will return 304 for the client to read from the cache. If they are different, it will return a new file to the client and update the value of the last modified response header field.

Advantages of using if modified since and last modified:

- When the cache is valid, the server will not return the file to the client, but directly return the 304 status code to let the client - get the file from the cache. It greatly saves traffic, bandwidth and server pressure.

Disadvantages of using if modified since and last modified:
- The last-Modified expiration time can only be accurate to seconds. If you both modify the file and get the file in the same second, the client cannot get the latest file.

### Etag & If-None-Match

In order to solve the problem that the file modification time can only be accurate to seconds, we introduce Etag response header. Etag is calculated from the file modification time and file size. The value of Etag will change only when the file content or modification time changes.

When the client requests the server for the first time, the server will return an Etag response header. When the client requests the server, it will bring the If-None-Match request header field. The value of this field is the value of Etag returned by the server. After receiving the request, the server will compare whether the two values are the same. If they are the same, it will return 304 for the client to read from the cache. If they are different, it will return a new file to the client and update the value of the Etag response header field.

Advantages of using Etag and If-None-Match:

- When the cache is valid, the server will not return the file to the client, but directly return the 304 status code to let the client get the file from the cache. It greatly saves traffic, bandwidth and server pressure.
- The problem of modifying and reading in one second is solved.

## Expansion

### Cache expires problem

The introduction of cache is a good thing, which can greatly improve the response speed and reduce the pressure on the server, but there will also be some problems, such as why the client still sees old files when we clearly update the system version. There are different solutions in different times.

- Old solution
In the old scheme, the file name is manually modified or the version number and timestamp are added after the file name, so that the client will request and use a new file, and the previous strong cache will become invalid even within the validity period.

```Javascript
<script src="http://randy.js?version=1.1.1> </script>
```

- New solution
In the current construction stage, there is basically no need for manual operation. It is built automatically using construction tools such as wbpack, Gulp, Grunt and so on. For example, when using webpack to build, the hash value will be automatically calculated according to the file name or file content to name the file. When the content or file name changes, the built file name will be different, which also solves the problem that the strong cache is still valid.

- pragma
Pragma is an old product and has been gradually abandoned. Some websites still keep this field for downward compatibility. When the value of pragma is no cache, it means that caching is disabled. The priority is pragma > cache control > expires.

- Flowchart
![image.png-98.8kB][1]
![image.png-224.6kB][2]

##Cache configuration

If we use nginx as the web server, we can configure it as follows

```Javascript
location / {

  # other config
  ...

  if ($request_uri ~* .*[.](js|css|map|jpg|png|svg|ico)$) {
    # cache 1 month for non html files
    add_header Cache-Control "public, max-age=2592000";
  }

  if ($request_filename ~* ^.*[.](html|htm)$) {
    # use negotiaion cache for html
    add_header Cache-Control "public, no-cache";
  }
}
```

## Where does cache store

According to the cache location, we can divide it into memory cache, disk cache and service worker. In the developer tool of chrome, we can see the final processing method of a request in the column of network - > size: if the size (k, m, etc.) indicates that it is a network request, otherwise it will list <span style="background-color:rgb(255, 245, 227)">from memory cache</span>, <span style="background-color:rgb(255, 245, 227)">from disk cache</span> <span style="background-color:rgb(255, 245, 227)">from serviceworker</span> means that the cache has been hit.

- memory cache: Is a cache in memory (in contrast, disk cache is a cache on the hard disk). According to the common sense of the operating system: read the memory first, and then the hard disk
- disk cache: HTTP cache, as its name suggests, is a cache stored on the hard disk, so it is persistent and actually exists in the file system. And it allows the same resources to be used across sessions or even across sites. For example, both sites use the same picture
- The above cache strategy and cache / read / invalidate actions are determined by the browser. We can only set some fields of the response header to tell the browser, but can't operate by ourselves. Service work gives us another more flexible and direct operation mode. We can find service workers from Chrome's application. This cache is permanent, that is, close the tab or browser, and it will still be opened next time (but the memory cache is not). There are two situations that will cause the resources in this cache to be cleared: manually calling API <span style="background-color:rgb(255, 245, 227)">cache.delete(resource)</span> or the capacity exceeds the limit and is completely emptied by the browser


  [1]: http://static.zybuluo.com/RandyGong/p7p9qijus29e8xxz6w6gzb6m/image.png
  [2]: http://static.zybuluo.com/RandyGong/fsvtcvp8wxqqs1fxo3c357lp/image.png