---
title: Network rules
description: Syntax for the network rules Zen supports to filter and modify HTTP traffic.
---

This document describes the syntax for the network rules Zen supports to filter and modify HTTP traffic.

## Top-level syntax

A rule must have a [match expression](#match-expressions) and may optionally include *modifiers*, separated from the match expression by the `$` character:

```AdBlock
<match-expression>
! or
<match-expression>$<modifiers>
```

A match expression determines whether a URL matches a rule.

If modifiers are absent from a rule, it simply blocks all requests whose URLs match. If modifiers are present, they perform additional matching against metadata (e.g., HTTP method), or modify requests or responses in various ways.

## Match expressions

URLs are matched against match expressions according to these rules:
- Percent-encoded triplets (e.g., `%20`) are left as-is and are not decoded into their canonical form.
- Internationalised domain names are processed in ASCII (Punycode-encoded and prefixed with `xn--`).

If the match expression starts and ends with `/`, it is interpreted as a [regexp-based expression](#regexp-based-expressions). Otherwise, it is interpreted as a [pattern-based expression](#pattern-based-expressions).

### Pattern-based expressions

**Pattern-based expressions** may contain any character that is valid in a URL (according to [RFC 3986 Section 2](https://www.rfc-editor.org/rfc/rfc3986#section-2)), as well as four special tokens: `*`, `||`, `^`, and `|`:

```AdBlock
.cgi?ad=
-adobe-analytics/
/ad_manager/*
||youtube.com/youtubei/v1/player
||s-cdn.anthropic.com/images/*.gif
|about
```

By default, pattern-based expressions can match starting at any position in the URL. The pattern must be matched in full, from the first to the last character.

Regular characters behave as expected:
- `a` matches any URL containing `a`
- `.` matches any URL containing `.`
- runs of consecutive characters are matched verbatim – e.g., `/path/example` matches any URL containing `/path/example` in any position

The special tokens function as follows:

#### `*`

Called a **wildcard**, it matches any number of characters, including none.

For example, `a*d` matches `abcd`, `a23d`, and `ad`.

#### `||`

Called a **domain boundary**, it matches domain and subdomain roots: the end of the scheme plus `://`, and subsequent `.` characters in the hostname portion of the URL.

For example, `||irbis.sh` matches all of the following:
- `https://irbis.sh`
- `wss://irbis.sh`
- `http://foo.bar.irbis.sh`

but not `https://github.com/?from=irbis.sh`.

#### `^`

Called a **separator**, it matches one or more characters other than a letter, digit, `_`, `-`, `.`, or `%`. It also matches the end of a URL.

For example, `/path^` matches `/path`, `/path/`, and `/path/?p`.

#### `|`

Called an **anchor**, it matches the beginning or the end of a URL.

For example, `.js|` matches `/script.js`, but not `/script.js?p`. `|https://irbis` matches `https://irbis.sh`, but not `https://github.com?from=https://irbis.sh`.

### Regexp-based expressions

**Regexp-based expressions** allow using [regular expressions](https://en.wikipedia.org/wiki/Regular_expression) to match rules against URLs:

```AdBlock
/ads\d+\.js/
/(https?:\/\/)104\.154\..{100,}/
```
> [!NOTE]
> Zen uses Go's standard library `regexp` package, which does not support some features occasionally used in regexp-based expressions, like backreferences. For an overview of the supported syntax, see [pkg.go.dev/regexp/syntax](https://pkg.go.dev/regexp/syntax). We also recommend testing the rules in advance via [regex101.com](https://regex101.com/) with the flavour set to Golang.

## Modifiers

A single rule may have multiple modifiers, separated by a comma: `script,method=get`.

Modifiers can be categorised into those that just declare their name, called *flag modifiers* (like `script`, `third-party`, `document`), and others that need an associated value, called *parametrised modifiers*, with the value specified after `=` (like `method=get`, `domain=example.com`, `removeparam=id`).

Additionally, some modifiers restrict when a rule applies; these are called *condition modifiers* (like `script`, `domain=...`, `method=get`). Others instruct a rule to perform an action other than plain blocking; these are called *action modifiers* (like `removeparam=...`, `removeheader=...`, `jsonprune=...`).

Some modifier sections include links to relevant unit tests. Go readers can refer to them for additional examples of behaviour.

### Condition modifiers

#### Content type modifiers

{{< badge "flag" >}}
{{< badge content="tests" color="blue" link="https://github.com/irbis-sh/zen-desktop/blob/master/internal/networkrules/rulemodifiers/contenttype_test.go" >}}

**Content type modifiers** match the requested resource type – script, stylesheet, font, image, etc. Zen determines the resource type using the [`Sec-Fetch-Dest`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest) request header.

Multiple content type modifiers may be present in a rule. At least one must match for the rule to be applied.

##### `font`

Matches if the request is for a font. Corresponds to [`Sec-Fetch-Dest: font`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#font).

##### `image`

Matches if the request is for an image. Corresponds to [`Sec-Fetch-Dest: image`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#image).

##### `media`

Matches if the request is for media content – video, audio, or a text track. Corresponds to [`Sec-Fetch-Dest: video`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#video), [`audio`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#audio), and [`track`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#track).

##### `object`

Matches if the request is for an object. Corresponds to [`Sec-Fetch-Dest: object`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#object).

##### `script`

Matches if the request is for a script. Corresponds to [`Sec-Fetch-Dest: script`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#script).

##### `stylesheet`

Matches if the request is for a CSS stylesheet. Corresponds to [`Sec-Fetch-Dest: style`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#style).

##### `subdocument`

Matches if the request is for a frame or an iframe. Corresponds to [`Sec-Fetch-Dest: frame`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#frame) and [`iframe`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#iframe).

##### `xmlhttprequest`

Matches if the request is made via [AJAX](https://developer.mozilla.org/en-US/docs/Glossary/AJAX). Corresponds to [`Sec-Fetch-Dest: empty`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#empty). May also be written as `xhr`.

##### `other`

Matches if the requested resource type corresponds to none of the other content type modifiers.

#### `domain`

{{< badge "parametrised" >}}
{{< badge content="tests" color="blue" link="https://github.com/irbis-sh/zen-desktop/blob/master/internal/networkrules/rulemodifiers/domain_test.go" >}}

Matches if the request is made from a specified domain, based on the value of the [`Referer`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Referer) header.

The modifier's value may include multiple expressions, separated by `|`. This is a logical OR: if any one expression matches, the modifier matches.

Each expression may be:

- A regular domain name, e.g., `domain=example.net`, `domain=irbis.sh`.
- A [functional second-level domain (SLD)](https://icannwiki.org/Second_Level_Domain) wildcard pattern, in the format `label.*`, where `*` corresponds to any [public suffix](https://publicsuffix.org/). E.g., `domain=example.*` matches `example.net`, `example.com`, but not `example.nested.domain.net`.
- A regular expression matching the domain name, starting and ending with `/`.

Expressions may be prefixed with `~`, which inverts the match (e.g., `domain=~example.net` matches any domain other than `example.net`). Either all expressions or none of them must be prefixed with `~`.

#### `header`

{{< badge "parametrised" >}}
{{< badge content="tests" color="blue" link="https://github.com/irbis-sh/zen-desktop/blob/master/internal/networkrules/rulemodifiers/header_test.go" >}}

Matches a response against a specified header.

The value may use one of two formats:

- A header name, matched case-insensitively. E.g., `header=set-cookie` matches any response that has the `Set-Cookie` header set.
- A header name, matched case-insensitively, followed by `:` and a header value expression, one of the following:
  - A full header value. E.g., `header=set-cookie:flavour=choco` matches any response with `Set-Cookie: flavour=choco`.
  - A regular expression. E.g., `header=set-cookie:/tt/` matches any response with the `Set-Cookie` header set, and the value matching `/tt/`.

#### `method`

{{< badge "parametrised" >}}
{{< badge content="tests" color="blue" link="https://github.com/irbis-sh/zen-desktop/blob/master/internal/networkrules/rulemodifiers/method_test.go" >}}

Matches the request's [HTTP method](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods).

The value may include multiple methods, separated by `|`. This is a logical OR: if any one matches, the modifier matches. Methods are matched case-insensitively, so `method=get` and `method=GET` are equivalent.

Methods may be prefixed with `~`, which inverts the match (e.g., `method=~get` matches any method other than `GET`). Either all methods or none of them must be prefixed with `~`.

#### `third-party`

{{< badge "flag" >}}

Matches if the request is *third-party* – made to a different site than the one the user is visiting. Zen determines this from the [`Sec-Fetch-Site`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Site) request header, treating a request as third-party when its value is `cross-site`.

May be inverted with `~third-party`, which instead matches *first-party* requests – those made to the same origin or site (`Sec-Fetch-Site: same-origin` or `same-site`).

### Action modifiers

> [!TIP]
> Action modifiers may be combined with condition modifiers to restrict where the transformation applies.

#### `removeparam`

{{< badge "flag" >}}
{{< badge "parametrised" >}}
{{< badge content="tests" color="blue" link="https://github.com/irbis-sh/zen-desktop/blob/master/internal/networkrules/rulemodifiers/removeparam_test.go" >}}

Removes query parameters from the request URL's query string.

The value may take one of several forms:

- Absent entirely: `removeparam` removes all query parameters.
- A parameter name: `removeparam=utm_source` removes the `utm_source` parameter.
- An inverted name, prefixed with `~`: `removeparam=~id` removes every parameter except `id`.
- A regular expression, starting and ending with `/`, matched against each parameter's full `name=value` string: `removeparam=/^utm_/` removes any parameter whose name begins with `utm_`. Appending `i` after the closing `/` makes the match case-insensitive.
- An inverted regular expression, prefixed with `~`: `removeparam=~/^id=/` removes every parameter whose `name=value` does *not* match.

#### `removeheader`

{{< badge "parametrised" >}}
{{< badge content="tests" color="blue" link="https://github.com/irbis-sh/zen-desktop/blob/master/internal/networkrules/rulemodifiers/removeheader_test.go" >}}

Removes a header from the response, or, with the `request:` prefix, from the request. Header names are matched case-insensitively.

- `removeheader=set-cookie` removes the `Set-Cookie` header from the response.
- `removeheader=request:x-client-data` removes the `X-Client-Data` header from the request.

For safety, a number of security- and protocol-critical headers cannot be removed – including [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS) headers, [`Content-Security-Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy), [`Strict-Transport-Security`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Strict-Transport-Security), `Content-Type`, and `Content-Length`. A rule targeting one of these is rejected when its filter list is parsed; the complete list lives in [`removeheader.go`](https://github.com/irbis-sh/zen-desktop/blob/master/internal/networkrules/rulemodifiers/removeheader.go).

#### `jsonprune`

{{< badge "parametrised" >}}
{{< badge content="tests" color="blue" link="https://github.com/irbis-sh/zen-desktop/blob/master/internal/networkrules/rulemodifiers/jsonprune_test.go" >}}

Removes elements from a JSON response body. Only responses with a `Content-Type` of `application/json` are affected; others are passed through unmodified.

The value is a [JSONPath](https://en.wikipedia.org/wiki/JSONPath) expression, and every element it selects is deleted. For example, `jsonprune=$.ads` removes the top-level `ads` field, and `jsonprune=$.items[*].ad` removes the `ad` field from every object in the `items` array. If the expression matches nothing, the response is left unchanged.

> [!NOTE]
> Zen parses the expression with the [ajson](https://github.com/spyzhov/ajson) library. Refer to [its documentation](https://github.com/spyzhov/ajson#jsonpath) for the supported JSONPath syntax.

#### `scramblejs`

{{< badge "parametrised" >}}
{{< badge content="tests" color="blue" link="https://github.com/irbis-sh/zen-desktop/blob/master/internal/networkrules/rulemodifiers/scramblejs_test.go" >}}

Replaces occurrences of one or more keys in JavaScript with unique, randomly generated identifiers. This applies to responses served as `text/javascript` or `application/javascript`, as well as inline `<script>` blocks in HTML documents. Text outside of scripts is left untouched.

The value is a `|`-separated list of keys, e.g., `scramblejs=key1|key2`. Keys are matched literally, not as regular expressions. Each occurrence is substituted with a fresh random identifier (a letter followed by alphanumeric characters), so code that relies on a shared, predictable name – for example, an anti-adblock flag read across several scripts – sees a different value in each place.

#### `remove-js-constant`

{{< badge "parametrised" >}}

Removes top-level JavaScript `var` declarations from script responses. This applies to responses served as `text/javascript`, as well as inline `<script>` blocks in HTML documents.

The value is a dot-separated path to the binding to remove. `remove-js-constant=adblock` removes a top-level `var adblock = ...` declaration entirely, while a longer path descends into object literals: `remove-js-constant=config.ads` removes only the `ads` property from a top-level `var config = { ... }`, leaving the rest of the object intact. Multiple paths may be specified, separated by `|`.

Only `var` declarations are processed; `let` and `const` bindings are left untouched.

## Hosts rules

Zen supports a subset of the hosts file syntax. If the IP address specified by the rule is `0.0.0.0` or `127.0.0.1`, Zen will block any traffic directed to the hostname. If the hostname is a domain name, **Zen will also block traffic to any subdomain**. Note that this differs from DNS resolver behaviour.

Zen also supports rules with an IP address followed by multiple hostnames.

For additional reference on hosts files, see [Wikipedia](https://en.wikipedia.org/wiki/Hosts_(file)) and [hosts(5)](https://man7.org/linux/man-pages/man5/hosts.5.html).

### Example

The rule

```AdBlock
0.0.0.0 irbis.sh
```

blocks all traffic to `irbis.sh` and all subdomains, including `www.irbis.sh` and `docs.irbis.sh`.
