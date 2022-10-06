# user-locale-client-hints explainer - DRAFT

This is the repository for user-locale-client-hints. You're welcome to
[contribute](CONTRIBUTING.md)!
## Authors:

- [Romulo Cintra](https://github.com/romulocintra)
- [Ujjwal Sharma](https://github.com/ryzokuken)

## Participate
- [GitHub repository](/)
- [Issue tracker](/issues)

## Introduction

User preferences, are often system-wide settings (such as in Android, macOS, or Windows). Operating systems allow the user to specify custom overrides for settings such as:

  * Hour cycle (24-hour or 12-hour time)
  * Calendar system
  * Measurement unit preferences (metric or imperial)
  * Date/time patterns
  * Number separators (comma or period)

However, there’s currently no reliable way to access this information from the Web Platform to help craft better user experiences. Allowing web developers to access this information would allow them to improve the accessibility and usability of their websites, and bring the user experience of web applications closer to that of native applications.


## Goals & Motivation


To allow web applications to get access to higher quality user preferences to assist Internationalization.

Client and server-side applications should access and share consistently locale user preferences that will help solve cases like:

> _Alice, an end user, reads en-US, but prefers using a 24-hour clock. She has set that preference in her operating system settings. Native apps are able to respect her preference. However, web apps cannot access OS user preferences, so on the Web Platform, Alice sees a 12-hour clock, the default for en-US._

For **_client side applications_**, the best way to get them would be through a browser API that fetches this information from the different platform-specific OS APIs.

For **_server side applications_**, one way to get access to this information would be via HTTP headers on the request.

![locale-hints-flow](https://user-images.githubusercontent.com/1572026/182210112-69fb7769-eafa-43ab-bad0-21eb49489d50.png)

## Proposed Solution

### Client Hints

A [HTTP Client Hint](https://datatracker.ietf.org/doc/html/rfc8942) is a request header field that is used by HTTP clients to indicate configuration data that can be used by the server to select an appropriate response. Each one conveys a list of user locale preferences that the server can use to adapt and optimize the response.

### Proposed Syntax

Servers will receive no information about the user's locale preferences. Servers can instead opt-into receiving such information via a new `Locale-Preferences` Client Hints.

We are proposing [BCP47](https://www.rfc-editor.org/rfc/bcp/bcp47.txt) extension subtags or compatible, as the mechanism for delivering user locale preferences. This allows the handle of preferences consistently across the industry, using as reference the Unicode Key Extensions in [UTS 35](https://unicode.org/reports/tr35/tr35.html#Key_And_Type_Definitions_) to define a baseline for the most common user preferences<sup>[*1](#1)<sup>. 

To accomplish this, Browsers should introduce several new `Client Hint` header fields where information can be obtained by using the list of headers below, which would represent the best intent of resolving the information using [Add Likely Subtags ](https://www.unicode.org/reports/tr35/#Likely_Subtags) algorithm and values set by the user in their user locale preferences. 


**`Sec-CH-Locale-Preferences`**

The  `Sec-CH-Locale-Preferences` header field represents the inline representation of locale preferences. For example:

```
// Language "en-US" - no user preferences set
Sec-CH-Locale-Preferences: "en-Latn-US"

// Language "en-US" - user set calendar, hour cycle and metric system
Sec-CH-Locale-Preferences: "en-Latn-US-u-ca-gregory-hc-h24-ms-uksystem"
```

**`Sec-CH-Locale-Preferences-u-`**

The following table represents the list of headers to opt-in for individual `Locale-Preferences`, using the `"-u-key"` suffix. 


| Client Hint | Example output | Allowed Values | Description |
| --- | --- | --- | --- |
| Sec-CH-Locale-Preferences-u-ca | `Sec-CH-Locale-Preferences-u-ca : "buddhist"`  |  | Calendar system / Calendar algorithm |
| Sec-CH-Locale-Preferences-u-cf | `Sec-CH-Locale-Preferences-u-cf: "account"`    | standard, account | Currency Format style, whether to use accounting currency format |
| Sec-CH-Locale-Preferences-u-co | `Sec-CH-Locale-Preferences-u-co: "search"`     | standard, search, phonetic... | Collation type, sort order |
| Sec-CH-Locale-Preferences-u-cu | `Sec-CH-Locale-Preferences-u-cu: "EUR"`        | 	ISO 4217 codes | Currency type |
| Sec-CH-Locale-Preferences-u-em | `Sec-CH-Locale-Preferences-u-em: "emoji"`      | emoji , text, default | Emoji presentation style |
| Sec-CH-Locale-Preferences-u-fw | `Sec-CH-Locale-Preferences-u-fw: "sun"`        | "sun", "mon" ... "sat" | First day of week  |
| Sec-CH-Locale-Preferences-u-hc | `Sec-CH-Locale-Preferences-u-hc: "h12"`        | h12 , h23, h11, h24 | Hour cycle, i.e., 12-hour or 24-hour clock |
| Sec-CH-Locale-Preferences-u-ms | `Sec-CH-Locale-Preferences-u-ms: "metric"`     | metric , ussystem , uksystem | Measurement system |
| Sec-CH-Locale-Preferences-u-nu | `Sec-CH-Locale-Preferences-u-nu: "latn"`       | Unicode script subtag(arabext...)    <br> | Numbering system |
| Sec-CH-Locale-Preferences-u-tz | `Sec-CH-Locale-Preferences-u-tz: "Atlantic/Azores"; "Atlantic/Madeira"; "Europe/Lisbon"` | Unicode short time zone IDs | Time zone |
| Sec-CH-Locale-Preferences-u-rg | `Sec-CH-Locale-Preferences-u-rg: "PT"` | [Unicode Region Subtag](https://unicode.org/reports/tr35/tr35.html#unicode_region_subtag) | Region override |

For example:

```
// Language "en-US" - no user preferences set
Sec-CH-Locale-Preferences-u-ca: ""
Sec-CH-Locale-Preferences-u-hc: ""

// Language "en-US" - user set calendar and hour cycle
Sec-CH-Locale-Preferences-u-ca: "gregory"
Sec-CH-Locale-Preferences-u-hc: "h12"
```

### Javascript API

These client hints should also be exposed via JavaScript APIs via `navigator.locales` as suggested in [#68](https://github.com/tc39/ecma402/issues/68) or by creating a new `navigator.localePreferences` that exposes `Locale-Preferences` information.



## Examples

1. The client makes an initial request to the server:
   ```
   GET / HTTP/1.1
   Host: example.com
   ```

2. The server responds, telling the client via `Accept-CH` that it accepts the
   `Sec-CH-Locale-Preferences` and the `Sec-CH-Locale-Preferences-fw` Client Hints.
   ```
   HTTP/1.1 200 OK
   Content-Type: text/html
   Accept-CH: Sec-CH-Locale-Preferences, Sec-CH-Locale-Preferences-fw
   ```

3. Then subsequent requests to https://example.com will include the following request headers in case the user sets `calendar` and `first day of week` values:
   ```
   GET / HTTP/1.1
   Host: example.com
   Sec-CH-Locale-Preferences: "en-u-ca-gregory-fw-sun"
   Sec-CH-Locale-Preferences-fw: "sun"
   ```
   
   In case the user did not set any `Locale-Preferences`, request headers have the best attempt of information if it's resolved or exists
   ```
   GET / HTTP/1.1
   Host: example.com
   Sec-CH-Locale-Preferences: "en-Latn-US"
   Sec-CH-Locale-Preferences-fw: ""
   ```

4. The server can then tailor the response to the client's preferences accordingly.

## Privacy and Security Considerations

There are some concerns that exposing this information would give trackers, advertisers and malicious web services another fingerprinting vector. That said, this information may or may not already be available to a certain extent to such services, based on the host and the user’s settings. So the use of `Sec-CH-` prefix is to forbid access to these headers containing `Locale Preferences` information from JavaScript, and demarcate them as browser-controlled client hints so they can be documented and included in requests without triggering CORS preflights.

Client Hints provides a powerful content negotiation mechanism that enables us to adapt content to users' needs without compromising their privacy. It does that by requiring server opt-in, which guarantees that access to the information requires active and tracable action on the server's side. As such, the mechanism does not increase the web's current active fingerprinting surface. The [Security Considerations](https://datatracker.ietf.org/doc/html/rfc8942#section-4) of HTTP Client Hints and the [Security Considerations](https://tools.ietf.org/html/draft-davidben-http-client-hint-reliability-02#section-5) of Client Hint Reliability likewise apply to this proposal.

## FAQ

**Q:** Does this proposal support non BCP-47 compatibile options ? 
   - **A:** *At the moment we are inclined to use BCP-47, but scalability it's important and a mechanism that allows non-BCP47 data to be available as user preference it's desirable, the `-u-key` prefix might enable ways for doing it*

**Q:** Aren’t you adding a lot of new headers? Isn’t that going to bloat requests?
   - **A:** *It’s true this proposal adds multiple new headers per request. But we don’t
expect every site to use or need all the hints for every request, and the `Sec-CH-Locale-Preferences` single header is able to provide most of the needed information.

**Q:** I have to parse `en-Latn-US-u-ca-gregory-cu-EUR-hc-h24-ms-uksystem` string to get 'hour cycle' value?   
- **A:** *You either parse and search wanted language tags or use individual `Locale-Preferences` options*

## References
- [Design Doc - User Preferences on the Web](https://docs.google.com/document/d/1YWkivRAR8OcKQqIqbdfi4_fksBjAdfGqUumpdCQ_aGs/edit#heading=h.2efe18287cds)
- [The Lang Client Hint](https://github.com/WICG/lang-client-hint)
- [Expose datetime formatting user preferences · Issue #38](https://github.com/tc39/ecma402/issues/38)
- [Add navigator.locales for user preferences · Issue #68](https://github.com/tc39/ecma402/issues/68)
- [Allow for implementations to retrieve settings from host environment · Issue #109](https://github.com/tc39/ecma402/issues/109) 
- [RFC: Add user preferences to HTTP header · Issue #416](https://github.com/tc39/ecma402/issues/416)
- [Region Override Support · Issue #370](https://github.com/tc39/ecma402/issues/370)
- [[Proposal] Make an API for locale negotiation · Issue #513](https://github.com/tc39/ecma402/issues/513) (`Intl.LocaleMatcher`)


[1] - To be discussed - related issues ([#3](https://github.com/tc39/proposal-intl-locale/issues/3), [#6](https://github.com/tc39/ecma402/issues/6), [#38](https://github.com/tc39/ecma402/issues/38), [#68](https://github.com/tc39/ecma402/issues/68), [#580](https://github.com/tc39/ecma402/issues/580))
