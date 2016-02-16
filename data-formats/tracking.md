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

Details of what must be collected at which event for stringified_json_data is explained in following sections. Version string starts from “1.0.0”, which is also the default value for logs from our old clients when version is omitted.

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

Most of these information would be of not much interest except for Perfect Price and therefore no specific action is necessary.


