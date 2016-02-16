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

. Details of what must be collected at which event for stringified_json_data is explained in following sections. For version, please use “1.2.0”, for now, while we may ask for changes if needed in the future.

