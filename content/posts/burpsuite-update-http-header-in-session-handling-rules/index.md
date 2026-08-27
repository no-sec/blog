---
title: BurpSuite – Update HTTP Header in Session Handling Rules
date: 2018-09-18
description: First, this post will not cover the basics of recoding macros or use of the session handling rules in BurpSuite
draft: false
tags:
  - BurpSuit
  - CSRF
  - Session Handling
toc: false
authors: [marko]
---


# BurpSuite – Update HTTP Header in Session Handling Rules

## Introduction

First, this post will not cover the basics of recoding macros or use of
the session handling rules in BurpSuite. There are a lot of basic stuff
to be found in the internet \[1\]\[2\]. However, by default, Burp is
unable to update or modify an http request HTTP header by using session
handling rules and macros. This can cause in problems if you investigate
REST APIs or applications which protects requests with one-time CSRF
tokens. Further, the Portswigger community blog is not very useful to
address this problem \[4\]\[5\].

## Solution

However, the portswigger github repo provides an easy solution. The
Readme.md describes the steps to configure the session handling rule to
update an http header using a crafted burp plugin. All information about
the implementation of the plugin is provided in the repo as well.
Summarizing, everything that needs to be done is to clone the custom
session token repository \[5\], update the global variable
*SESSION_ID_KEY* in the *BurpExtender* class, build and load the Burp
plugin. Quite easy.

Happy hacking!

\[1\]
[https://digi.ninja/blog/burp_macros.php](https://digi.ninja/blog/burp_macros.php "https://digi.ninja/blog/burp_macros.php")  
\[2\]
[https://www.blackhillsinfosec.com/using-simple-burp-macros-to-automate-testing/](https://www.blackhillsinfosec.com/using-simple-burp-macros-to-automate-testing/ "https://www.blackhillsinfosec.com/using-simple-burp-macros-to-automate-testing/")  
\[3\]
[https://support.portswigger.net/customer/portal/questions/16076238-update-header-in-session-handling-macros](https://support.portswigger.net/customer/portal/questions/16076238-update-header-in-session-handling-macros "https://support.portswigger.net/customer/portal/questions/16076238-update-header-in-session-handling-macros")  
\[4\]
[https://support.portswigger.net/customer/portal/questions/11698880-how-do-i-change-a-http-header-value-for-active-scan-with-stored-state-file](https://support.portswigger.net/customer/portal/questions/11698880-how-do-i-change-a-http-header-value-for-active-scan-with-stored-state-file "https://support.portswigger.net/customer/portal/questions/11698880-how-do-i-change-a-http-header-value-for-active-scan-with-stored-state-file")–  
\[5\]
[https://github.com/PortSwigger/example-custom-session-tokens](https://github.com/PortSwigger/example-custom-session-tokens "https://github.com/PortSwigger/example-custom-session-tokens")
