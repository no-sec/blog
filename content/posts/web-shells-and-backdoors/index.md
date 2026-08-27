---
title: Web Shells and Backdoors
date: 2016-08-31
description: In April 2016 researchers of the Stony Brook University and Ruhr-University Bochum published a study about (malicious) PHP web shells with the title “No Honor among Thieves A Large-Scale Analysis of Malicious Web Shells”.
draft: false
tags:
  - General
toc: false
authors: [glia]
---

# Web Shells and Backdoors

## Introduction

In April 2016 researchers of the Stony Brook University and
Ruhr-University Bochum published a study about (malicious) PHP web
shells with the title “No Honor among Thieves: A Large-Scale Analysis of
Malicious Web Shells”. \[1\] Their goal was to analyze how many PHP web
shells contain backdoors or other malicious functions not in the
interest of the user. That has to be seen regardless of the fact that
web shells are often used for malicious activities themselves.

Here we summarize their findings which in our opinion are of interest
for the wider audience in security and penetration test as web shells
are regularly used for testing purposes. The authors also provide
insight into other areas of web shell internals and more details on some
mechanisms used in regard to malicious behavior. If you are interested,
we encourage you to read the full paper (see references section).

The researchers collected a larger sample of PHP web shells on the
internet which they divided into a static and a dynamic set. This
separation was done with the intention to conduct a static source code
analysis to identify and extract functions of interest on the one hand
and to observe dynamic behavior (i.e. actual network communication) in a
honeypot scenario they did setup on the other hand. In the latter case
the authors monitored client-side and server-side traffic so they could
capture any communications attempt with 3^(rd) parties.

The researchers came up with a sample size of 481 in the static set and
541 in the dynamic set totaling to 1022 web shell samples. These rather
high numbers include customizations as well as different versions of
well-known web shells of the families c99, r57, WSO, B347k, NST, NCC and
Crystal.

## Results

First off, the researchers indeed found that a significant portion of
inspected web shells include backdoors or otherwise unwanted functions.
The study isn’t entirely precise on the actual number as will be
discussed in more detail later. Thus we only give the upper and lower
bounds of affected web shells.

The authors define three different functions they classify as malicious:
Authentication bypass, homephoning and unintended server side
activities. All are described in the chapters below. The following table
summarizes their findings of malicious web shells grouped by the total,
static and dynamic sets:

[TABLE]

1: Number of web shells within the static set which have authentication
functionality.

### Authentication bypass

If the (legit) user uploads a web shell to a server he has the interest
to protect access to it by employing mandatory authentication. If the
shell is of malicious nature it might contain an authentication bypass
function which effectively allows a 3^(rd) party access to the web shell
without knowledge of the authentication credentials. This bypass might
range from hardcoded (hidden) credentials, bypass procedures to hidden
(unprotected) functionalities.

The authors used the static result set of web shells to analyze
bypasses. Out of the 481 web shells in the static set, 250 provide
authentication mechanism in the first place. Those are the ones of
interest since the others don’t offer authentication at all.

Out of those 250, **30.8% (77) can be bypassed** which are **16% of the
total number (481)** of web shells in the static result set.

### Homephoning

These are functionalities which inform a 3^(rd) party about the
existence of a web shell deployment. It tries to inform the 3^(rd) party
about an infected URL/web server, possibly also containing further
information about authentication credentials or details about the
initial user. Homephoning mechanisms can either be implemented from the
server side (PHP code sends out the information) or client side
(JavaScript or HTML resource loading from the users browsers).

Here the authors used a dynamic analysis and therefore the result set
with 541 web shells. Out of those **29.2% (~158) performed client-side**
homephoning attempts and **4.8% (~26) performed server-side**
homephoning.

### Server Side Activity

The idea here is that malicious web shells might implement functions
which directly perform malicious activities on the infected server such
as downloading additional payloads or manipulation of the operating
system. To asses these kinds of activities the researchers monitored the
execution of specific PHP function calls within each PHP web shell.
Among those where functions like “fopen(), file_get_contents(), exec(),
system(), etc.

They found that out of the dynamic result set **31.4% (~170) *invoke*
monitored functions**.

### Further Results: Anti-Virus Detection Rates

While sitting on a larger sample set of web shells the researchers
tested the static result set (consisting of 481 web shells) on the
online service “VirusTotal” which tests any given sample with common
anti-virus scanners.

They found that the **most accurate anti-virus scanner detected 72.5%
(~349)** of the uploaded shells as such while **in 90% (~433) of web
shells at least one scanner** out of many detected the sample. This
finding underlines that using anti-virus scanners alone are not a
sufficient protection against malware.

## Discussion

### About the results

The research gives some structured information about the risks
associated with the usage of PHP web shells. A significant percentage of
all reviewed samples perform at least one kind of malicious function.
This shows that there is substantial risk of using public web shells for
legit penetration testing purposes. If for example one uses a backdoored
web shell during a client’s penetration test it basically opens the
system to external adversaries potentially putting the whole
infrastructure at risk.

However there are scenarios in which a penetration tester might still
want to use a web shell to dig further into a tested network or system
to identify even more weaknesses in the infrastructure. While it might
be tempting, given the researchers results it might not be a good idea
to use a “random-well-known-internet-web-shell”. The risk to introduce a
backdoored web shell is relatively high.

Considering this, at the very least a web shell should be intensively
reviewed for malicious functions before deployment. But manually
reviewing and testing can be a lot of work and many ways exists for a
skilled programmer to hide malicious functions within a PHP script. So
the risk of oversights is quite high. Given that it is probably best to
“simply” write your own web shell which only contains the needed
functions and is guaranteed to be free of any kind of backdoor.
Additionally you can take measures yourself to protect the web shell
installation in the client\`s and your own interest.

### About the study

Although the research gives interesting facts about the risk of public
web shell usage a few aspects should be kept in mind when reading
through the numbers and facts. First the researchers are not entirely
clear about their process of collecting the sample web shells. Consider
the following two quotes from the original study:

Quotes taken from chapter 3 – Data Collection:

“***We also removed shells which we knew would not provide any
interesting****, from a static analysis point of view, **results**, such
as shells that were just printing local system information or **were
merely sending an email to a predefined email address**. Finally, since
we use static analysis to discover popular features of web shells, **we
removed shells which did not conform to our model of what a shell is**,
such as shells which were just **connecting to a remote server**, in a
**bot-like fashion**, and **executing any commands** which that server
would return.*”

“*For the dynamic analysis part, we employed a more forgiving approach
where shells were not removed unless they were completely broken, i.e.,
**we kept the shells whose rendering in a browser would result in some
sort of visible UI**, even if that was partially broken. \[…\] **We
removed shells which provided no obvious malicious functionality**, such
as files calling the  phpinfo() function and exiting.*”

The first quote is a bit confusing: Basically the authors say they
rejected some web shell samples because they either send only e-mails,
connected to a remote server, acted like a bot or executed commands on a
remote server. In the way this is stated it sounds like the described
functions are actually out of the category “malicious behavior” and it
is not clear why the authors excluded those shells. Also they don’t give
a detailed description of what their used model of a malicious web shell
is, which would have been helpful.

In the second quote the authors state that they also used web shells
which did not properly render, i.e. which might contain errors inside
the PHP script and terminate somewhere during execution. Why those have
been kept inside the dynamic result set is not clear. Especially
regarding their selection process of the static result set they state:
“\[…\] we only used the shells that were either fully syntactically
correct or the ones which we could repair with a reasonable amount of
manual effort.” This is quite the contrary of what you might expect:
Using a broken PHP script for execution and using a functional correct
one for static analysis / no execution?

A second problem with the study is that the static and dynamic result
sets might overlap but are not identical. Yet they are analyzed
independently. Therefore, in the way the results are presented a clear
picture of how many different web shell are malicious cannot be given.
For example: a web shell having an authentication bypass backdoor might
also implement homephoning functions on the server and the client side.
This specific shell would therefore be counted in each sub-set of
malicious shells. This is why we initially only gave a range of possible
overall percentages of malicious web shells. Here it would be helpful if
the authors would further clarify on their results leading to more
precise assessment of the actual risk.

## References

\[1\] O. Starov, J. Dahse, S. S. Ahmad and T. Holz. [No Honor among
Thieves: A Large-Scale Analysis of Malicious Web
Shells](https://www.securitee.org/files/webshells_www2016.pdf). 25th
International World Wide Web Conference (WWW), Montreal, April 2016.
