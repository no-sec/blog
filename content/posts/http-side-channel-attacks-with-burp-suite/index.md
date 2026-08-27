---
title: HTTP Side-Channel Attacks with Burp Suite
date: 2016-04-25
description: Ever wanted a simple tool capable of performing a side-channel attackof HTTP requests? A common use case would be to test of the login action takes more time of the provided username is known by the system compared to an unknown username. 
draft: false
tags:
  - BurpSuit
  - Side-Channel
toc: false
authors: [SecMyth]
---
# HTTP Side-Channel Attacks with Burp Suite

Ever wanted a simple tool capable of performing a [side-channel
attack](https://en.wikipedia.org/wiki/Side-channel_attack) of HTTP
requests? A common use case would be to test of the login action takes
more time of the provided username is known by the system compared to an
unknown username. The theory behind such an attack is that the code
performing the login stops more early in the flow if the used username
is not present in the system.

Well, to test such a weakness it is necessary to perform certain HTTP
requests, e.g. logins, and to capture the need time to receive a
response. But what tools are fulfilling these requirements? The search
can’t be this hard. In fact the most common web application penetration
tool [Burp Suite](https://portswigger.net/burp/) is in fact capable of
this task. The following steps will demonstrate how to conduct a test
for side-channel attacks.

1.  Select the Request to be analysed and send it to the intruder  
    ![Send Request to
    Intruder](./Burp_sidechannel_1.png)
2.  In the Intruder Tab you have to select the Payload Position (marked
    with §§) you want to alternate, e.g. the username:  
    ![Mark Username with
    §§](./Burp_sidechannel_2.png)
3.  Configure the rest of the Intruder Options as you like, while you at
    least have to choose a payload
4.  After this you can start the attack. The intruder will fire one http
    request for each configured payload item.
5.  The new window listing all issued requests has to be configured to
    list the time needed to receive the response:  
    ![Adjust the Columns in the Intruder
    results](./Burp_sidechannel.png)
6.  With this extra column it is now possible to see if certain username
    influence the response time and if the login is vulnerable to a
    side-channel attack:  
    ![Overview of the overall needed time of each
    request](./Burp_sidechannel_4.png)

## Important remark on those tests on HTTP traffic:

- At first it is recommended to send the same payload multiple times to
  verify that the measures time is reproducible.
- Don’t bomb the server with requests. If you are using multiple threads
  to send these requests, the server has to deal with all of them at the
  same time, which reduces the validity of the measurement. Leave the
  server enough time to recover from each request.
- The time to complete the response does not only depend on the code
  residing on the server but mostly on the route to the server and back.
  So most likely any measures done to remote servers are not
  representative as the network component has a much higher impact on
  the response time. The result is that all responses will take almost
  the same time to complete.

 
