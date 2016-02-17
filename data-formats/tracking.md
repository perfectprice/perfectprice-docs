# Introduction
In this document, “The Company” refers to the company that signs up for Perfect Price, Inc. service, i.e. a client company of Perfect Price, Inc.

### Common Log Format
For certain events, we collect logs in JSON format, which is transferred to our log server as a POST request. Any data sent must be in the following form :

```json
{
    "json_text": stringified_json_data,
    "version": version
}
```

where :

| Field | Description |
| -------------: |:------------- |
| stringified_json_data | a Javascript dictionary variable that contains all data we collected first converted into JSON object and second converted into a string |
| version | a log version string in “integer.integer.integer” format |

Details of what must be collected at which event for stringified\_json\_data is explained in following sections. Version string starts from “1.0.0”, which is also the default value for logs from our old clients when version is omitted.

### Trasmitting Logs

Once constructed, make a POST request to :

```
http://log.pfpr.co/log/97da7ff7f0e157b0874d839cfd79876e
```

Response for this call is :

```json
{
    "status": 0,
    "message": "Success",
    "data": {
        "ppu": "e87e63b6f59a5ec6e0dc6bf472988d78",
        "time": "1446411036"
    }
}

```

where

| Field | Description |
| -------------: |:------------- |
| status | an integer valued status code, which is 0 if POST request is successfully processed, or non-zero values if there was any backend error. Details of possible error status codes and what must be done is explained in the following section. |
| message | a string containing a brief note about what happened. |
| data | additional meta information : |
| data/ppu | a unique hash assigned to this user, which is usually added into the cookie for key “ppu”. |
| data/time | a timestamp that records logging time. |


### Handling Errors##### Connection ErrorsIf our log server at log.pfpr.co is not responding or connectable, then logs sent will not be saved in our backend. Unless such logs are queued either at browser or some other server and resent, logs will be permanently lost. We our internal alerting on to avoid any service down time, so mostly this shouldn’t happen, but for safely, it is recommended to implement any actions to avoid log loss as much as possible. If such connection error persists consistently, please email us immediately at info@perfectprice.io.##### Backend ErrorsAll backend errors happened in log.pfpr.co, while sending logs is internally handled so no specific error handling is necessary from the browser side.##### Frontend ErrorsAny frontend errors except for connection exceptions must be handled appropriately to avoid any log loss. Unless tracking Javascript code is provided by Perfect Price, Inc., this is the responsibility of The Company. # Log DataThis section covers specifics of stringified\_json\_data explained in previous section.### Common Log Data FormatFor any log sent at page load event, stringified\_json\_data must be of this format, e.g. :```json{    "document": {        "base URI": "https://www.betabrand.com/",        "cookie": "",        "document URI": "https://www.betabrand.com/",        "domain": "www.betabrand.com",        "title": "Betabrand - Pants, Jackets, Hoodies, Breakthroughs",        "url": "https://www.betabrand.com/"    },    "id": "97da7ff7f0e157b0874d839cfd79876e",    "navigator": {        "app code name": "Mozilla",        "app name": "Netscape",        "app version": "5.0 (Macintosh; Intel Mac OS X 10_10_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2490.80 Safari/537.36",        "cookie enabled": true,        "language": "en-US",        "online": true,        "platform": "MacIntel",        "product": "Gecko",        "user agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2490.80 Safari/537.36"    },    page_specific_data,    "screen": {        "available height": 832,        "available width": 1440,        "color depth": 24,        "height": 900,        "pixel depth": 24,        "width": 1440    },    "type": type_of_log,    "window": {        "closed": false,        "default status": "",        "inner height": 286,        "inner width": 1270,        "outer height": 832,        "outer weight": 1270,        "screen x": 22,        "screen y": 23,        "status": ""    }}```
where page\_specific\_data and type\_of\_log differ on different pages and events. For single page apps, page view events include not only the initial page load, but also subsequent pages re-rendering due to clicks. For each of a list of specified such page views in single page apps, event must be sent by The Company, or Perfect Price, Inc. if our tracking Javascript is used.