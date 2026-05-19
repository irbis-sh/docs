---
title: Network rules
description: Syntax for the network rules Zen supports to filter and modify HTTP traffic.
---

This document describes the syntax for the network rules Zen supports to filter and modify HTTP traffic.

> [!INFO]
> This document is being actively worked on. Check back later for more.

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

Modifiers can be categorised into those that just declare their name, called *flag modifiers* (like `script`, `third-party`, `document`), and others that need an associated value, called *parametrised modifiers* (like `method=get`, `domain=example.com`, `removeparam=id`).

Additionally, some modifiers restrict when a rule applies; these are called *condition modifiers* (like `script`, `domain=...`, `method=get`). Others instruct a rule to perform an action other than plain blocking; these are called *action modifiers* (like `removeparam=...`, `removeheader=...`, `jsonprune=...`).

### Content type modifiers

{{< badge "flag" >}}
{{< badge "condition" >}}

**Content type modifiers** match the requested resource type – script, stylesheet, font, image, etc. Zen determines the resource type using the [`Sec-Fetch-Dest`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest) request header.

Multiple content type modifiers may be present in a rule. At least one must match for the rule to be applied.

#### font

Matches if the request is for a font. Corresponds to [`Sec-Fetch-Dest: font`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#font).

#### image

Matches if the request is for an image. Corresponds to [`Sec-Fetch-Dest: image`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#image).

#### media

Matches if the request is for media content – video, audio, or a text track. Corresponds to [`Sec-Fetch-Dest: video`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#video), [`audio`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#audio), and [`track`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#track).

#### object

Matches if the request is for an object. Corresponds to [`Sec-Fetch-Dest: object`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#object).

#### script

Matches if the request is for a script. Corresponds to [`Sec-Fetch-Dest: script`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#script).

#### stylesheet

Matches if the request is for a CSS stylesheet. Corresponds to [`Sec-Fetch-Dest: style`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#style).

#### subdocument

Matches if the request is for a frame or an iframe. Corresponds to [`Sec-Fetch-Dest: frame`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#frame) and [`iframe`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#iframe).

#### xmlhttprequest

Matches if the request is made via [AJAX](https://developer.mozilla.org/en-US/docs/Glossary/AJAX). Corresponds to [`Sec-Fetch-Dest: empty`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-Fetch-Dest#empty).

## Hosts rules

Zen supports a subset of the hosts file syntax. If the IP address specified by the rule is `0.0.0.0` or `127.0.0.1`, Zen will block any traffic directed to the hostname. If the hostname is a domain name, **Zen will also block traffic to any subdomain**. This differs from DNS resolver behaviour.

Zen also supports rules with an IP address followed by multiple hostnames.

For additional reference on hosts files, see [Wikipedia](https://en.wikipedia.org/wiki/Hosts_(file)) and [hosts(5)](https://man7.org/linux/man-pages/man5/hosts.5.html).

### Example

The rule

```AdBlock
0.0.0.0 irbis.sh
```

blocks all traffic to `irbis.sh` and all subdomains, including `www.irbis.sh` and `docs.irbis.sh`.
