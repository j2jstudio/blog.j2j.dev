---
title: "Content Blocker Filters 规则"
date: 2021-04-20T14:20:36+08:00
draft: true
tags:
  - iOS
  - Content Blocker Extension
---


参考：

* [https://webkit.org/blog/3476/content-blockers-first-look/](https://webkit.org/blog/3476/content-blockers-first-look/)

*  [https://developer.apple.com/documentation/safariservices/creating_a_content_blocker](https://developer.apple.com/documentation/safariservices/creating_a_content_blocker)

* [https://help.eyeo.com/en/adblockplus/how-to-write-filters](https://help.eyeo.com/en/adblockplus/how-to-write-filters)
  


A trigger dictionary must include a url-filter key, which specifies a pattern to match the URL against. The remaining keys are optional and modify the behavior of the trigger. For example, you can limit the trigger to specific domains or have it not apply when a match is found on a specific domain.

For example, to write a trigger for image and stylesheet resources found on any domain except those specified:

```
"trigger": {
        "url-filter": ".*",
        "resource-type": ["image", "style-sheet"],
        "unless-domain": ["your-content-server.com", "trusted-content-server.com"]
}
```


<table data-v-4906e9b4=""><thead data-v-4906e9b4=""><tr data-v-4906e9b4=""><th data-v-4906e9b4="" scope="col"><p data-v-4906e9b4="">Syntax</p></th><th data-v-4906e9b4="" scope="col"><p data-v-4906e9b4="">Description</p></th></tr></thead><tbody data-v-4906e9b4=""><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">.*</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">Matches all strings with a dot appearing zero or more times. Use this syntax to match every URL.</p></td></tr><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">.</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">Matches any character.</p></td></tr><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">\.</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">Explicitly matches the dot character.</p></td></tr><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">[a-b]</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">Matches a range of alphabetic characters.</p></td></tr><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">(abc)</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">Matches groups of the specified characters.</p></td></tr><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">+</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">Matches the preceding term one or more times.</p></td></tr><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">*</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">Matches the preceding character zero or more times.</p></td></tr><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">?</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">Matches the preceding character zero or one time.</p></td></tr></tbody></table>



<table data-v-4906e9b4=""><thead data-v-4906e9b4=""><tr data-v-4906e9b4=""><th data-v-4906e9b4="" scope="col"><p data-v-4906e9b4="">Trigger</p></th><th data-v-4906e9b4="" scope="col"><p data-v-4906e9b4="">Description</p></th></tr></thead><tbody data-v-4906e9b4=""><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">url-filter-is-case-sensitive</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">A Boolean value. The default value is false.</p></td></tr><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">if-domain</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">An array of strings matched to a URL's domain; limits action to a list of specific domains. Values must be lowercase ASCII, or punycode for non-ASCII. Add <code data-v-7fb764c1="" data-v-4906e9b4="">*</code> in front to match domain and subdomains. Can't be used with <code data-v-7fb764c1="" data-v-4906e9b4="">unless-domain</code>.</p></td></tr><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">unless-domain</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">An array of strings matched to a URL's domain; acts on any site except domains in a provided list. Values must be lowercase ASCII, or punycode for non-ASCII. Add <code data-v-7fb764c1="" data-v-4906e9b4="">*</code> in front to match domain and subdomains. Can't be used with<code data-v-7fb764c1="" data-v-4906e9b4=""> if-domain</code>.</p></td></tr><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">resource-type</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">An array of strings representing the resource types (how the browser intends to use the resource) that the rule should match. If not specified, the rule matches all resource types. Valid values: <code data-v-7fb764c1="" data-v-4906e9b4="">document</code>, <code data-v-7fb764c1="" data-v-4906e9b4="">image</code>, <code data-v-7fb764c1="" data-v-4906e9b4="">style-sheet</code>, <code data-v-7fb764c1="" data-v-4906e9b4="">script</code>, <code data-v-7fb764c1="" data-v-4906e9b4="">font</code>, <code data-v-7fb764c1="" data-v-4906e9b4="">raw</code> (Any untyped load), <code data-v-7fb764c1="" data-v-4906e9b4="">svg-document</code>, <code data-v-7fb764c1="" data-v-4906e9b4="">media</code>, <code data-v-7fb764c1="" data-v-4906e9b4="">popup</code>, <code data-v-7fb764c1="" data-v-4906e9b4="">ping</code>.</p></td></tr><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">load-type</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">An array of strings that can include one of two mutually exclusive values. If not specified, the rule matches all load types. <code data-v-7fb764c1="" data-v-4906e9b4="">first-party </code>is triggered only if the resource has the same scheme, domain, and port as the main page resource. <code data-v-7fb764c1="" data-v-4906e9b4="">third-party </code>is triggered if the resource is not from the same domain as the main page resource.</p></td></tr><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">if-top-url</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">An array of strings matched to the entire main document URL; limits the action to a specific list of URL patterns. Values must be lowercase ASCII, or punycode for non-ASCII. Can't be used with <code data-v-7fb764c1="" data-v-4906e9b4="">unless-top-url</code>.</p></td></tr><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">unless-top-url</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">An array of strings matched to the entire main document URL; acts on any site except URL patterns in provided list. Values must be lowercase ASCII, or punycode for non-ASCII. Can't be used with <code data-v-7fb764c1="" data-v-4906e9b4="">if-top-url</code>.</p></td></tr></tbody></table>


Add Actions to Your Content Blocker
When a trigger matches a resource, the browser queues the associated action for execution. Safari evaluates all the triggers, it executes the actions in order. When a domain matches a trigger, all rules after the triggered rule that specify the same action are skipped.

Group the rules with similar actions together to improve performance. For example, first specify rules that block content loading, followed by rules that block cookies. The trigger evaluation continues at the first rule that specifies a different action.

There are only two valid fields for actions: type and selector. An action type is required. If the type is css-display-none, a selector is required as well; otherwise the selector is optional.

For example, you can specify the follow type and selector:

```
"action": {
        "type": "css-display-none",
        "selector": "#newsletter, :matches(.main-page, .article) .news-overlay"
}
```


<table data-v-4906e9b4=""><thead data-v-4906e9b4=""><tr data-v-4906e9b4=""><th data-v-4906e9b4="" scope="col"><p data-v-4906e9b4="">Syntax</p></th><th data-v-4906e9b4="" scope="col"><p data-v-4906e9b4="">Description</p></th></tr></thead><tbody data-v-4906e9b4=""><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">block</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">Stops loading of the resource. If the resource was cached, the cache is ignored.</p></td></tr><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">block-cookies</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">Strips cookies from the header before sending to the server. Only cookies otherwise acceptable to Safari's privacy policy can be blocked. Combining with <code data-v-7fb764c1="" data-v-4906e9b4="">ignore-previous-rules</code> doesn't override the browser’s privacy settings.</p></td></tr><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">css-display-none</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">Hides elements of the page based on a CSS selector. A <code data-v-7fb764c1="" data-v-4906e9b4="">selector</code> field contains the selector list. Any matching element has its <code data-v-7fb764c1="" data-v-4906e9b4="">display</code> property set to <code data-v-7fb764c1="" data-v-4906e9b4="">none</code>, which hides it.</p></td></tr><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">ignore-previous-rules</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">Ignores previously triggered actions.</p></td></tr><tr data-v-4906e9b4=""><td data-v-4906e9b4=""><p data-v-4906e9b4=""><code data-v-7fb764c1="" data-v-4906e9b4="">make-https</code></p></td><td data-v-4906e9b4=""><p data-v-4906e9b4="">Changes a URL from <code data-v-7fb764c1="" data-v-4906e9b4="">http</code> to <code data-v-7fb764c1="" data-v-4906e9b4="">https</code>. URLs with a specified (nondefault) port and links using other protocols are unaffected.</p><p data-v-4906e9b4=""></p></td></tr></tbody></table>


For the selector field, specify a string that defines a selector list. This value is required when the action type is css-display-none. If it's not, the selector field is ignored by Safari. Use CSS identifiers as the individual selector values, separated by commas. Safari and WebKit supports all of its CSS selectors for Safari content-blocking rules.


## ADBLOCK PLUS 过滤规则介绍


### 过滤规则的类型

* 阻止型过滤器(Blocking filters) 应用于网络层，用于判断该请求 request 是否应该被阻止。
* 内容过滤器 (Content filters) 用于隐藏页面中的元素 
* 例外过滤器 (Exception filters) 用于忽略特定请求、内容的过滤，类似于白名单

### 基本过滤规则

基于关键字的过滤（所有请求的路径包含该关键字都会被屏蔽）, 如 http://example.com/ads/banner123.gif 将屏蔽该图片。其中 123 是个随机数字，屏蔽单个地址经常会不起作用，可以使用通配符编写更通用的过滤规则。如 http://example.com/ads/banner*.gif （屏蔽ads目录下所有前缀为banner的gif图片）或 http://example.com/ads/（屏蔽所有ads目录下的内容）。

```
Note: 不可以使用太多的通配符， 比如 http://example.com/ 将会屏蔽所有的banner，同时该规则将会屏蔽所有 example.com 下的其他内容。
```

### 例外规则 

