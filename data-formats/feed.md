# 1. Google Product Feeds

### Introduction

Many companies already have data feeds ready to send to Google or other shopping search engines or services. Sometimes, Perfect Price can simply import data in this format, simplifying integration.

### What to include

If you have an export job already configured, you are welcome to start by sending that so we can analyze if it has what we need. Specific fields of interest are listed below, but it is usually better to be overly inclusive. Any heiarchy information and identifiers (UPC, etc.) if available are of utmost importance. 

### Where to send it

Please send to your Perfect Price representative directly, and they will get it processed by the engineering team.

### Format

We follow the Google specification located here: 

https://support.google.com/merchants/answer/188494?hl=en

Notes on special fields:

| Field | Notes |
|-------------:|:-------------|
| __gtin__ | This field is very important for finding products at competitors' sites. Be sure to include if available. | 
| __mpn__ | This field is also important for finding products at competitors' sites. Be sure to include if available. |
| __item_group_id__ | While primiarly useful for advertising, this structure is also useful to Perfect Price, if available. If categories and heirarchy is represented elsewhere, this is otpional. | 
| __Shipping__ | Not necessary but can be included if desired |
| __Tax__ | Not necessary but can be included if desired |

### Cost

Cost data may be included this feed, following the spec for Cost outlined in [Custom Data](custom.md) in Perfect Price documentation. 

