---
title: Tools for debugging HTTP client request and its TLS settings
categories: http-client http-request tls
---

Debugging HTTP client requests or HTTP client TLS settings is crucial, especially when you don't own the target server. Without access to the server logs, you cannot check what is going wrong with your client's request or its TLS settings.

There are two websites which I use as tools:
- [httpbin.org](https://httpbin.org) for debugging HTTP client requests.
- [www.howsmyssl.com](https://www.howsmyssl.com/a/check) for checking HTTP client's TLS settings.

## httpbin.org
This is created by [Kenneth Reitz](https://kennethreitz.org). The website provide API can showing all the request information like `parameters`, `body content` and `headers`. To use this, just fire the request to the corresponding endpoint and it will return the request information as the response content.

`https://httpbin.org/anything` is one of the endpoint showing most of the request information. Please refer to [httpbin.org](https://httpbin.org) for more information about other endpoints.

**Request example**
``` bash
curl -i --compressed \
-H "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:127.0) Gecko/20100101 Firefox/127.0" \
-H "Accept-Language: en-US,en;q=0.5" \
-H "Accept: text/html,application/xhtml+xml,application/xml" \
https://httpbin.org/anything
```

**Response header**
```
HTTP/2 200
date: Wed, 26 Jun 2024 10:19:16 GMT
content-type: application/json
content-length: 549
server: gunicorn/19.9.0
access-control-allow-origin: *
access-control-allow-credentials: true
```

**Response content(IP has been masked here)**
``` json
{
  "args": {},
  "data": "",
  "files": {},
  "form": {},
  "headers": {
    "Accept": "text/html,application/xhtml+xml,application/xml",
    "Accept-Encoding": "deflate, gzip, br",
    "Accept-Language": "en-US,en;q=0.5",
    "Host": "httpbin.org",
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:127.0) Gecko/20100101 Firefox/127.0",
    "X-Amzn-Trace-Id": "Root=1-667beb24-44b1c2063308e1d20b4b4c76"
  },
  "json": null,
  "method": "GET",
  "origin": "xxx.xxx.xxx.xxx",
  "url": "https://httpbin.org/anything"
}
```

## www.howsmyssl.com
This website is useful for checking your client's TLS settings such as `TLS version` and `available cipher suites`. To use this, just fire the request to the endpoint [https://www.howsmyssl.com/a/check](https://www.howsmyssl.com/a/check) and it will return your client TLS information as the response content.

**Request example**
``` bash
curl -i https://www.howsmyssl.com/a/check
```

**Response header**
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Connection: close
Content-Length: 1459
Content-Type: application/json
Strict-Transport-Security: max-age=631138519; includeSubdomains; preload
Vary: Accept-Encoding
Date: Wed, 26 Jun 2024 10:33:14 GMT
```

**Response content**
``` json
{
    "given_cipher_suites": [
        "TLS_AES_256_GCM_SHA384",
        "TLS_CHACHA20_POLY1305_SHA256",
        "TLS_AES_128_GCM_SHA256",
        "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384",
        "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384",
        "TLS_DHE_RSA_WITH_AES_256_GCM_SHA384",
        "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256",
        "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256",
        "TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256",
        "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256",
        "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
        "TLS_DHE_RSA_WITH_AES_128_GCM_SHA256",
        "TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384",
        "TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384",
        "TLS_DHE_RSA_WITH_AES_256_CBC_SHA256",
        "TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256",
        "TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256",
        "TLS_DHE_RSA_WITH_AES_128_CBC_SHA256",
        "TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA",
        "TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA",
        "TLS_DHE_RSA_WITH_AES_256_CBC_SHA",
        "TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA",
        "TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA",
        "TLS_DHE_RSA_WITH_AES_128_CBC_SHA",
        "TLS_RSA_WITH_AES_256_GCM_SHA384",
        "TLS_RSA_WITH_AES_128_GCM_SHA256",
        "TLS_RSA_WITH_AES_256_CBC_SHA256",
        "TLS_RSA_WITH_AES_128_CBC_SHA256",
        "TLS_RSA_WITH_AES_256_CBC_SHA",
        "TLS_RSA_WITH_AES_128_CBC_SHA",
        "TLS_EMPTY_RENEGOTIATION_INFO_SCSV"
    ],
    "ephemeral_keys_supported": true,
    "session_ticket_supported": false,
    "tls_compression_supported": false,
    "unknown_cipher_suite_supported": false,
    "beast_vuln": false,
    "able_to_detect_n_minus_one_splitting": false,
    "insecure_cipher_suites": {},
    "tls_version": "TLS 1.3",
    "rating": "Probably Okay"
}
```

You can see my curl client is using `TLS 1.3` from the response. Below is another example to try with TLS 1.2.

**Request example with specify TLS 1.2 in client**
``` bash
curl -i --tlsv1.2 --tls-max 1.2 https://www.howsmyssl.com/a/check
```

**Response header**
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Connection: close
Content-Length: 1378
Content-Type: application/json
Strict-Transport-Security: max-age=631138519; includeSubdomains; preload
Vary: Accept-Encoding
Date: Wed, 26 Jun 2024 10:35:57 GMT
```

**Response content**
``` json
{
    "given_cipher_suites": [
        "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384",
        "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384",
        "TLS_DHE_RSA_WITH_AES_256_GCM_SHA384",
        "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256",
        "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256",
        "TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256",
        "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256",
        "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
        "TLS_DHE_RSA_WITH_AES_128_GCM_SHA256",
        "TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384",
        "TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384",
        "TLS_DHE_RSA_WITH_AES_256_CBC_SHA256",
        "TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256",
        "TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256",
        "TLS_DHE_RSA_WITH_AES_128_CBC_SHA256",
        "TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA",
        "TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA",
        "TLS_DHE_RSA_WITH_AES_256_CBC_SHA",
        "TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA",
        "TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA",
        "TLS_DHE_RSA_WITH_AES_128_CBC_SHA",
        "TLS_RSA_WITH_AES_256_GCM_SHA384",
        "TLS_RSA_WITH_AES_128_GCM_SHA256",
        "TLS_RSA_WITH_AES_256_CBC_SHA256",
        "TLS_RSA_WITH_AES_128_CBC_SHA256",
        "TLS_RSA_WITH_AES_256_CBC_SHA",
        "TLS_RSA_WITH_AES_128_CBC_SHA",
        "TLS_EMPTY_RENEGOTIATION_INFO_SCSV"
    ],
    "ephemeral_keys_supported": true,
    "session_ticket_supported": false,
    "tls_compression_supported": false,
    "unknown_cipher_suite_supported": false,
    "beast_vuln": false,
    "able_to_detect_n_minus_one_splitting": false,
    "insecure_cipher_suites": {},
    "tls_version": "TLS 1.2",
    "rating": "Probably Okay"
}
```

You can see my curl client is using `TLS 1.2` from the response.
