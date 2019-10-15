---
title: "2019 04 27 Fix Hugo Warning"
date: 2019-04-27T14:02:19+08:00
description: ""
draft: true
tags: []
categories: []
---

Building sites … WARN 2019/04/25 13:19:46 Page's .Hugo is deprecated and will be removed in a future release. Use the global hugo function.
WARN 2019/04/25 13:19:46 Page's .RSSLink is deprecated and will be removed in a future release. Use the Output Format's link, e.g. something like:
{{ with .OutputFormats.Get "RSS" }}{{ .RelPermalink }}{{ end }}.
WARN 2019/04/25 13:19:46 Page's .URL is deprecated and will be removed in a future release. Use .Permalink or .RelPermalink. If what you want is the front matter URL value, use .Params.url.

<!--more-->
