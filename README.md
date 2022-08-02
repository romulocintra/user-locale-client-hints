# user-locale-client-hints explainer

This is the repository for user-locale-client-hints. You're welcome to
[contribute](CONTRIBUTING.md)!
## Authors:

- [Romulo Cintra](@romulocintra)
- [Ujjwal Sharma](@ryzokuken)

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

We are inclined to use [BCP47](https://www.rfc-editor.org/rfc/bcp/bcp47.txt) mechanism for delivering user locale  preferences, allowing the handle of these preferences in a consistent way across the industry and with [UTS 35](https://unicode.org/reports/tr35/tr35.html#Key_And_Type_Definitions_) to define a set of the most common user preferences.

To accomplish this, Browsers should introduce several new Client Hint header fields:

**Minimize Locale Preferences**

The  `Sec-CH-Locale-Preferences` header field represents a minimum viable set of extension subtags resulting from the [Remove Likely Subtags ](https://www.unicode.org/reports/tr35/#Likely_Subtags) algorithm and overwritten values set by the user locale preferences. For example:


```
// Language "en-US" - no user preferences set
Sec-CH-Locale-Preferences: "en"
```

```
// Language "en-US" - user set calendar and hour cycle
Sec-CH-Locale-Preferences: "en-u-ca-gregory-hc-h12"
```

**Maximize Locale Preferences**

Granular information can be obtained  by using the list of headers bellow , they represent the best intent of resolving the information from [Add Likely Subtags ](https://www.unicode.org/reports/tr35/#Likely_Subtags) algorithm and by values set by the user in their user locale preferences. 


| Client Hint | Example output | Allowed Values | Description |
| --- | --- | --- | --- |
| Sec-CH-Locale-Preferences-u-ca | mul-u-ca-buddhist |  | Calendar system / Calendar algorithm |
| Sec-CH-Locale-Preferences-u-cf | mul-u-cf-account | standard, account | Currency Format style, whether to use accounting currency format |
| Sec-CH-Locale-Preferences-u-co | mul-u-cf-search | standard, search, phonetic... | Collation type, sort order |
| Sec-CH-Locale-Preferences-u-cu |  | 	ISO 4217 codes | Currency type |
| Sec-CH-Locale-Preferences-u-em |  | emoji , text, default | Emoji presentation style |
| Sec-CH-Locale-Preferences-u-fw |  | "sun", "mon" ... "sat" | First day of week  |
| Sec-CH-Locale-Preferences-u-hc |  | h12 , h23, h11, h24 | Hour cycle, i.e., 12-hour or 24-hour clock |
| Sec-CH-Locale-Preferences-u-ms |  | metric , ussystem , uksystem | Measurement system, i.e., metric or imperial |
| Sec-CH-Locale-Preferences-u-nu |  | Unicode script subtag(arabext...)    <br> | Numbering system |
| Sec-CH-Locale-Preferences-u-tz |  | Unicode short time zone IDs | Time zone |
| Sec-CH-Locale-Preferences-u-rg |  | [Unicode Region Subtag](https://unicode.org/reports/tr35/tr35.html#unicode_region_subtag) | Region override |


These client hints should also be exposed via JavaScript APIs via a new `navigator.localePreferences` attribute:

```
// TODO CODE
```

## [API 1]

[For each related element of the proposed solution - be it an additional JS method, a new object, a new element, a new concept etc., create a section which briefly describes it.]

```js
// Provide example code - not IDL - demonstrating the design of the feature.

// If this API can be used on its own to address a user need,
// link it back to one of the scenarios in the goals section.

// If you need to show how to get the feature set up
// (initialized, or using permissions, etc.), include that too.
```

[Where necessary, provide links to longer explanations of the relevant pre-existing concepts and API.
If there is no suitable external documentation, you might like to provide supplementary information as an appendix in this document, and provide an internal link where appropriate.]

[If this is already specced, link to the relevant section of the spec.]

[If spec work is in progress, link to the PR or draft of the spec.]

## [API 2]

[etc.]

## Key scenarios

[If there are a suite of interacting APIs, show how they work together to solve the key scenarios described.]

### Scenario 1

[Description of the end-user scenario]

```js
// Sample code demonstrating how to use these APIs to address that scenario.
```

### Scenario 2

[etc.]

## Detailed design discussion

### [Tricky design choice #1]

[Talk through the tradeoffs in coming to the specific design point you want to make.]

```js
// Illustrated with example code.
```

[This may be an open question,
in which case you should link to any active discussion threads.]

### [Tricky design choice 2]

[etc.]

## Considered alternatives

[This should include as many alternatives as you can,
from high level architectural decisions down to alternative naming choices.]

### [Alternative 1]

[Describe an alternative which was considered,
and why you decided against it.]

### [Alternative 2]

[etc.]

## Stakeholder Feedback / Opposition

[Implementors and other stakeholders may already have publicly stated positions on this work. If you can, list them here with links to evidence as appropriate.]

- [Implementor A] : Positive
- [Stakeholder B] : No signals
- [Implementor C] : Negative

[If appropriate, explain the reasons given by other implementors for their concerns.]

## References & acknowledgements

[Your design will change and be informed by many people; acknowledge them in an ongoing way! It helps build community and, as we only get by through the contributions of many, is only fair.]

[Unless you have a specific reason not to, these should be in alphabetical order.]

Many thanks for valuable feedback and advice from:

- [Person 1]
- [Person 2]
- [etc.]
