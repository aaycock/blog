---
title:  "Virtual SmartZone (vSZ) API Workaround"
date:   2021-02-13
categories: [ruckus, wifi]
tags: [ruckus, wifi]
---

A client recently asked for help with an odd issue that was preventing a clean sync of WLAN configurations for a site with more than 100 APs.

One of the first things I do in my CWAP-inspired troubleshooting process is to _capture and analyze the data_ about the configuration using the REST API, because the web UI doesn't always tell the full story.

On one of the API calls, I saw an error that looked like a parsing issue.

API call that fails:

```
get('/rkszones/f77a8816-3049-40cd-8484-82919275ddc3/apgroups/8b451880-993d-4fc3-a6ff-7814d016c111')
opening connection to 10.144.120.11:7443...
opened
starting SSL for 10.144.120.11:7443...
SSL established, protocol: TLSv1.2, cipher: ECDHE-RSA-AES128-SHA256
<- "GET /api/public/v8_2/rkszones/f77a8816-3049-40cd-8484-82919275ddc3/apgroups/8b451880-993d-4fc3-a6ff-7814d016c111 HTTP/1.1\r\nContent-Type: application/json;charset=UTF-8\r\nAccept: application/json\r\nCookie: JSESSIONID=70CDCC7B477BB0C770BFFCE2CA58D558\r\nAccept-Encoding: gzip;q=1.0,deflate;q=0.6,identity;q=0.3\r\nUser-Agent: Ruby\r\nConnection: close\r\nHost: 10.144.120.11:7443\r\n\r\n"
-> "HTTP/1.1 500 \r\n"
-> "Server: nginx\r\n"
-> "Date: Fri, 18 Sep 2020 20:47:56 GMT\r\n"
-> "Content-Type: application/json;charset=UTF-8\r\n"
-> "Transfer-Encoding: chunked\r\n"
-> "Connection: close\r\n"
-> "Cache-Control: no-store\r\n"
-> "X-XSS-Protection: 1; mode=block\r\n"
-> "X-Frame-Options: SAMEORIGIN\r\n"
-> "X-Content-Type-Options: nosniff\r\n"
-> "Content-Language: en-US\r\n"
-> "Content-Encoding: gzip\r\n"
-> "Vary: Accept-Encoding\r\n"
-> "\r\n"
-> "65\r\n"
reading 101 bytes...
-> "\x1F\x8B\b\x00\x00\x00\x00\x00\x00\x00\xAAV\xCAM-.NLOU\xB2Rr\xCB/R\xC8\xCC+(-Q(.)\xCA\xCCK\xB7R\x88Q\x8AQR\xD2QJ-*\xCA/r\xCEO\x01*2\x80\xF2B*\v@Z<\xF3JR\x8B\xF2\x12s\x14\x8AS\x8B\xCAR\x8B\x14"
read 101 bytes
reading 2 bytes...
-> "\r\n"
read 2 bytes
-> "0\r\n"
-> "\r\n"
Conn close
=> {"message"=>"For input string: \"\"", "errorCode"=>0, "errorType"=>"Internal server error"}
```

This error was being reported on the controller, so I dug into the logs and found this: 

Here is the error from the log:

```
2020-10-23 19:42:20,912 Web[wsg-exec-26] ERROR c.r.s.a.e.ExceptionControllerAdvice  - Exception: 
java.lang.NumberFormatException: For input string: ""
	at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65) ~[na:1.8.0_181]
	at java.lang.Integer.parseInt(Integer.java:592) ~[na:1.8.0_181]
	at java.lang.Integer.parseInt(Integer.java:615) ~[na:1.8.0_181]
	at com.ruckuswireless.scg.api.service.APGroupBS.getRadio50(APGroupBS.java:350) ~[scg-public-api-5.1.2-SNAPSHOT.jar:na]
	at com.ruckuswireless.scg.api.service.APGroupBS.fromDomainEntity(APGroupBS.java:588) ~[scg-public-api-5.1.2-SNAPSHOT.jar:na]
  ...
  ```

I wasn't sure if it was related at first, but I was able to reproduce it on-demand when I made my API calls, so I knew that I had found the culprit. 

The `getRadio50` method is the clue we are looking for. There is an empty string where a value is expected. These are the radio50 settings for the **AP group** we are looking for:

  ![vsz-toggle](/images/vsz-toggle.png)

To force a value, I toggled the settings to override, hit save, then toggled them back, and hit save again.

That did the trick! I was able to successfully make the API calls I needed to gather the WLAN/AP config data our WNMS needed to sync cleanly.