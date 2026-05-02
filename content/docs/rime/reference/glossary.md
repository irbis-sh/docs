---
title: Glossary
weight: 10
description: Definitions of key Zen Enterprise terms used throughout the platform and documentation.
---

Find out what key terms mean in the context of Zen Enterprise Edition.

- __Organization__<br />
  A company or group that uses Zen Enterprise Edition on its devices.
- __Dashboard__<br />
  The web-based management interface where administrators can configure and manage their organization's Zen Enterprise deployment. Available at [dash.irbis.sh](https://dash.irbis.sh).
- __Policy__<br />
  A collection of settings that govern how Zen behaves on managed devices. At the moment, policies contain _filter lists_, with more capabilities coming soon.
- __Filter list__<br />
  A set of rules that define which web requests and content to block, allow, or modify. Filter lists are plain text files hosted at a specified URL. Zen supports multiple filter list syntaxes, including:
  - [hosts files](https://en.wikipedia.org/wiki/Hosts_(file))
  - [EasyList](https://easylist.to)
  - [uBlock Origin](https://github.com/gorhill/uBlock/wiki/Static-filter-syntax)
  - [AdGuard](https://adguard.com/kb/general/ad-filtering/create-own-filters/)
- __Deployment token__<br />
  A unique identifier used during the _endpoint_ registration process to associate the _endpoint_ with a specific organization. Deployment tokens can carry multiple policies, which are automatically assigned to an _endpoint_ upon registration.
- __Endpoint__<br />
  A device (such as a computer or laptop) managed by an _organization_ through Zen Enterprise. Each endpoint runs the _agent_, which enforces the _policies_ assigned to it. Endpoint _policies_ can be updated remotely from the _dashboard_.
- __Agent__<br />
  The software component installed on each _endpoint_ that applies and enforces the policies assigned to it.
