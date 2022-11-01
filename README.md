# Explainer:  User Locale Preferences

## Table of Contents

- [Explainer:  User Locale Preferences](#explainer--user-locale-preferences)
  - [Table of Contents](#table-of-contents)
  - [Authors:](#authors)
  - [Participate](#participate)
  - [Introduction](#introduction)
  - [Use Cases & Motivation](#use-cases--motivation)
  - [Proposed Solution](#proposed-solution)
    - [Client Hints](#client-hints)
      - [Proposed Syntax](#proposed-syntax)
      - [Example](#example)
    - [Javascript API](#javascript-api)
      - [Proposed Syntax](#proposed-syntax-1)
      - [Examples](#examples)
  - [Privacy and Security Considerations](#privacy-and-security-considerations)
  - [Stakeholder Feedback / Opposition](#stakeholder-feedback--opposition)
  - [FAQ](#faq)
  - [References](#references)


## Authors:

- [Romulo Cintra](https://github.com/romulocintra)
- [Ujjwal Sharma](https://github.com/ryzokuken)

## Participate
- [GitHub repository](/)
- [Issue tracker](/issues)

## Introduction

User preferences are often system-wide settings (such as in Android, macOS, or Windows). Operating systems allow the user to specify custom overrides for settings such as:

  * Hour cycle (24-hour or 12-hour time)
  * Calendar system
  * Measurement unit preferences (metric or imperial)
  * Date/time patterns
  * Number separators (comma or period)

However, there’s currently no reliable way to access this information from the Web Platform to help craft better user experiences. Allowing web developers to access this information would allow them to improve the accessibility and usability of their websites, and bring the user experience of web applications closer to that of native applications.


## Use Cases & Motivation


To allow web applications to get access to higher quality user preferences to assist Internationalization.

Client and server-side applications should access and share consistently locale user preferences that will help solve cases like:

> _Alice, an end user, reads en-US but prefers using a 24-hour clock. She has set that preference in her operating system settings. Native apps are able to respect her preference. However, web apps cannot access OS user preferences, so on the Web Platform, Alice sees a 12-hour clock, the default for en-US._

> _other use cases: NodeJS programs that want to respect the user's hourCycle preference by passing it in to Intl.DateTimeFormat, Other non browser environments._

For **_client side applications_**, the best way to get them would be through a browser API that fetches this information from the different platform-specific OS APIs.

For **_server-side applications_**, one way to get access to this information would be via HTTP headers on the request.

![locale-hints-flow](https://user-images.githubusercontent.com/1572026/182210112-69fb7769-eafa-43ab-bad0-21eb49489d50.png)

## Proposed Solution

We propose to address the above use cases by using a group of [`Client Hints`](#client-hints) headers and a homologous [Javascript API](#javascript-api), both will be responsible for exposing and negotiating the exchange of user preferences from the OS to the required environment.

We are proposing [Unicode Extensions for BCP 47](https://cldr.unicode.org/index/bcp47-extension) or compatible as the main reference for the base mechanism for delivering user locale preferences. The goal is to allow the handling of user preferences consistently across the industry.



So, we will define a new standard `Locale-Preferences` Client Hints and `navigator.localePreferences`, that would map the user locale preferences using the following steps:

  1.  Validate if there is any fingerprinting mechanism and if OS preferences are allowed to be exposed
  2.  Read the available OS preferences
  3.  For each value compare it against the default  value for the given locale from ICU/CLDR or a list of user preferences to compare against
  4.  If the _user preference_ value differs, return the value
  5.  If the  _user preference_ value is the same, or not set, return the default value for the given locale


The following table suggests common user preferences [#416](https://github.com/tc39/ecma402/issues/416#issue-574957588) to be used, and does the correlation from  `Locale-Preferences` Client Hints and `navigator.localePreferences` to extension keys if they exist, or other values in case table is extended by user demand.

| Locale Preferences Name | Extension Key/Unicode Source | Example Values | Description |
| ----------------------- | -------| ------ |-----|
| "calendar"              |  `ca`  | buddhist, chinese...| [Calendar system / Calendar algorithm](https://github.com/unicode-org/cldr/blob/main/common/bcp47/calendar.xml) | 
| "currencyFormat"        |  `cf`  | standard, account | Currency Format style, whether to use accounting currency format |
| "collation"             |  `co`  | standard, search, phonetic... | Collation type, sort order |
| "currencyCode"          |  `cu`  | 	ISO 4217 codes | Currency type |
| "emojiStyle"            |  `em`  | emoji , text, default | Emoji presentation style |
| "firstDayOfTheWeek"      |  `fw`  | "sun", "mon" ... "sat" | First day of week  |
| "hourCycle"             |  `hc`  | h12 , h23, h11, h24 | Hour cycle, i.e., 12-hour or 24-hour clock |
| "measurementSystem"     |  `ms`  | metric , ussystem , uksystem | Measurement system |
| "measurementUnit"       |  `mu`  | celsius , kelvin , fahrenhe | [Measurement units currently only temperature](https://github.com/unicode-org/cldr/blob/7942fd82f7e673eb0b27f0bf2025a31572135d95/common/bcp47/measure.xml#L16) |
| "numberingSystem"       |  `nu`  | Unicode script subtag(arabext...)    <br> | Numbering system |
| "timeZone"              |  `tz`  | Unicode short time zone IDs | Time zone |
| "region"                |  `rg`  | [Unicode Region Subtag](https://unicode.org/reports/tr35/tr35.html#unicode_region_subtag) | Region override |


Other user preferences that might be included based on user research about OS preferences inputs and [#1308329](https://bugzilla.mozilla.org/show_bug.cgi?id=1308329), most of them don't have a 1:1 match with BCP47 extension keys, however, they are available in ICU/CLDR
 - number separator (grouping/decimal)
 - date formats short/medium/long/full
 - time formats short/medium/long/full
 - dayperiod names
 - time display with/without seconds
 - flash the time separators
 - show dayPeriod
 - show the day of the week
 - show date

> Note: The table and list are recommendations, they need to be validated and agreed by security/privacy teams and other stakeholders, but from now on using them as a reference for the proposal.

### Client Hints

A [HTTP Client Hint](https://datatracker.ietf.org/doc/html/rfc8942) is a request header field that is used by HTTP clients to indicate configuration data that can be used by the server to select an appropriate response. It defines an `Accept-CH` response header that servers can use to advertise their use of request headers for proactive content negotiation. Each new client hint conveys a list of user locale preferences that the server can use to adapt and optimize responses.


#### Proposed Syntax

Servers will receive no information about the user's locale preferences. Servers can instead opt-into receiving such information via a new `Locale-Preferences` Client Hints.

To accomplish this, Browsers should introduce several new `Client Hint` header fields as part of a [Structured Header](https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-header-structure) sent over request whose value is a list of defined user locale preferences.

**`Sec-CH-Locale-Preferences-*`**

The following table represents the list of headers returning individual opted-in `Locale-Preferences`. For example:


| Client Hint | Example output |
| --- | --- |
| Sec-CH-Locale-Preferences-calendar          | `Sec-CH-Locale-Preferences-calendar         : "buddhist"`  |  
| Sec-CH-Locale-Preferences-currencyFormat    | `Sec-CH-Locale-Preferences-currencyFormat   : "account"`    | 
| Sec-CH-Locale-Preferences-collation         | `Sec-CH-Locale-Preferences-collation        : "search"`     | 
| Sec-CH-Locale-Preferences-currencyCode      | `Sec-CH-Locale-Preferences-currencyCode     : "EUR"`        | 
| Sec-CH-Locale-Preferences-emojiStyle        | `Sec-CH-Locale-Preferences-emojiStyle       : "emoji"`      | 
| Sec-CH-Locale-Preferences-firstDayOfTheWeek  | `Sec-CH-Locale-Preferences-firstDayOfTheWeek : "sun"`        | 
| Sec-CH-Locale-Preferences-hourCycle         | `Sec-CH-Locale-Preferences-hourCycle        : "h12"`        | 
| Sec-CH-Locale-Preferences-measurementSystem | `Sec-CH-Locale-Preferences-measurementSystem: "metric"`     | 
| Sec-CH-Locale-Preferences-measurementUnit   | `Sec-CH-Locale-Preferences-measurementUnit  : "kelvin"`       | 
| Sec-CH-Locale-Preferences-numberingSystem   | `Sec-CH-Locale-Preferences-numberingSystem  : "latn";` | 
| Sec-CH-Locale-Preferences-timeZone          | `Sec-CH-Locale-Preferences-timeZone         : "Atlantic/Azores";`  |
| Sec-CH-Locale-Preferences-region            | `Sec-CH-Locale-Preferences-region           : "PT"` |
| Sec-CH-Locale-Preferences-dateFormat        | `Sec-CH-Locale-Preferences-dateFormat       : "EEE, d MMM yyyy HH:mm:ss Z"` |



#### Example
1. The client makes an initial request to the server:

```http
GET / HTTP/1.1
Host: example.com
```

2. The server responds, telling the client via an `Accept-CH` header (Section 2.2.1 of [[!RFC8942]]) along with the initial response with `Sec-CH-Locale-Preferences-numberingSystem` and the `Sec-CH-Locale-Preferences-timeZone` Client Hints: 

```http
HTTP/1.1 200 OK
Content-Type: text/html
Accept-CH: Sec-CH-Locale-Preferences-numberingSystem, Sec-CH-Locale-Preferences-timeZone
```

3. Then subsequent requests to https://example.com will include the following request headers in case the user sets `numberingSystem` and `timeZone` values:

```http
GET / HTTP/1.1
Host: example.com
Sec-CH-Locale-Preferences-calendar:"buddhist"
Sec-CH-Locale-Preferences-timeZone: "Africa/Lagos"
```

In case the user did not set any `Locale-Preferences` for accepted values, request headers return the default value for given locale
```http
GET / HTTP/1.1
Host: example.com
Sec-CH-Locale-Preferences-calendar:"gregory"
Sec-CH-Locale-Preferences-timeZone: "Europe/London"
```

4. The server can then tailor the response to the client's preferences accordingly.


### Javascript API

These client hints should also be exposed as JavaScript APIs via `navigator.locales` as suggested in [#68](https://github.com/tc39/ecma402/issues/68) or by creating a new `navigator.localePreferences` that exposes `Locale-Preferences` information as bellow.


#### Proposed Syntax

This might be written like so:

```js

navigator.localePreferences.calendar(); // =>  "gregory"
navigator.localePreferences.currencyFormat(); // => "EUR"
navigator.localePreferences.timeZone(); // =>  "Europe/London"
navigator.localePreferences.region(); // => "GB"
 
// user has not set `firstDayOfTheWeek` value in their OS, it returns the default value for given locale
navigator.localePreferences.firstDayOfTheWeek(); // =>  "7"
```

#### Examples

Use the `navigator.localePreferences` to populate data with user preferences

```js
// User set locale preferences for calendar , region and hourCycle
navigator.localePreferences.calendar(); // =>  "gregory"
navigator.localePreferences.region(); // => 'GB'
navigator.localePreferences.hourCycle(); // => 'h11';
navigator.localePreferences.timeZone(); // =>  'Europe/London'

const localePreferences =  (...iterate over needed navigator.localePreferences )


new Intl.Locale('es' ,  localePreferences ).maximize();
// => Locale {..., calendar:"gregory" , region:"GB" , hourCycle:"h11" }

new Intl.DateTimeFormat('es', localePreferences).resolvedOptions();
// =>  {locale: 'es', calendar: 'gregory', numberingSystem: 'latn', timeZone: 'Europe/London', ...}

```


## Privacy and Security Considerations

There are some concerns that exposing this information would give trackers, advertisers and malicious web services another fingerprinting vector. That said, this information may or may not already be available to a certain extent to such services, based on the host and the user’s settings. So the use of `Sec-CH-` prefix is to forbid access to these headers containing `Locale Preferences` information from JavaScript, and demarcate them as browser-controlled client hints so they can be documented and included in requests without triggering CORS preflights.

Client Hints provides a powerful content negotiation mechanism that enables us to adapt content to users' needs without compromising their privacy. It does that by requiring server opt-in, which guarantees that access to the information requires active and tracable action on the server's side. As such, the mechanism does not increase the web's current active fingerprinting surface. The [Security Considerations](https://datatracker.ietf.org/doc/html/rfc8942#section-4) of HTTP Client Hints and the [Security Considerations](https://tools.ietf.org/html/draft-davidben-http-client-hint-reliability-02#section-5) of Client Hint Reliability likewise apply to this proposal.




---- 

## Stakeholder Feedback / Opposition

- Brave : 
- Chrome : 
- Edge : 
- Firefox : 
- Safari : 

## FAQ

**Q:** Does this proposal support non Unicode Extension Key compatible options? 	
   - **A:**  *The first approach used a `-u` compatible syntax that could match 1:1 [Unicode Extension Keys](https://www.unicode.org/reports/tr35/#Key_And_Type_Definitions_), which would ease the interoperability, but at the same time limit future extensibility and scalability when non-BCP47 data would be available as user preference, so the table maps user preferences to BCP-47 compatible keys and future other sources.*

**Q:** Aren’t you adding a lot of new headers? Isn’t that going to bloat requests?
   - **A:** *It’s true this proposal adds multiple new headers per request. But we don’t expect every site to use or need all the hints for every request, and the `Sec-CH-Locale-Preferences` single header is able to provide most of the needed information.*

**Q:** How about `en-Latn-US-u-ca-gregory-cu-EUR-hc-h24-ms-uksystem` ? 
- **A:** *At the moment there is no support to convert to or from the Unicode format*

**Q:** How about a header that gives access to all user local preferences `Sec-CH-Locale-Preferences-all` `Sec-CH-Locale-Preferences-: calendar="gregory"; timeZone="Europe/London" ... `? 
- **A:** *There are use cases to simplify how to access this data but at the same time this would add new fingerprinting vectors*

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
