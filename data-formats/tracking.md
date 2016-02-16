# Introduction
In this document, “The Company” refers to the company that signs up for Perfect Price, Inc. service, i.e. a client company of Perfect Price, Inc.

### Common Log Format
For certain events, we collect logs in JSON format, which is transferred to our log server as a POST request. Any data sent must be in the following form :

```
{
    "json_text": stringified_json_data,
    "version": version
}
```
