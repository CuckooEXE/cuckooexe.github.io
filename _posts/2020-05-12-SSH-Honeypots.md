---
layout: post
title: SSH Honeypots
excerpt_separator: <!--more-->
---

Decided to create two honeypots ([instructions here](https://gist.github.com/AxelPersinger/912c74bc06466c143cab8ea01923d8d8)): 

* `honeypot-1.axelp.io` which is running SSH on port 22
* `honeypot-2.axelp.io` which is running SSH on port 7843

The primary reason I wanted to do this was to study the volume of attempted logins on a standard versus non-standard port over time. I have both servers reporting their `/var/log/auth.log` to a server. See the results below:

<!--more-->

## Failed SSH Logins

<iframe height="636" width="100%" frameborder="0" src="https://splunk.axelp.io:8000/en-US/embed?s=%2FservicesNS%2Fadmin%2Fsearch%2Fsaved%2Fsearches%2FFailed%2520SSH%2520Logins&oid=jMLRFcWwmpoI7_jAGZZZIqeA4tqayiRE3jgad2Q%5ECwFIxzQyLdLjA6PMuihKr%5Efg6afkUEf11v4hakCD7p0_tn1GcAUNpW4X%5EEJ_1a3N0D6AGrCvmWnVK9GtplklPPtbNwQXvnKJTCPV4Ndl"></iframe>

## Failed SSH Logins over Time, by Host

<iframe height="400" width="100%" frameborder="0" src="https://splunk.axelp.io:8000/en-US/embed?s=%2FservicesNS%2Fadmin%2Fsearch%2Fsaved%2Fsearches%2FFailed%2520SSH%2520Logins%2520over%2520Time&oid=dg96NeJtXFl19lpgBerb5l9oexG2ZNu0TZPqfpNoDu8U0nHLtQQm9vOlSl38YHqZn5EU%5ENwp%5EiftZNpXx17diYDZVrnXjOCLnwC_APhLl49JoSVkLxIGs8R8hzYyGLLFUVc3yamvmeS6dbPmyaS0cC7lrVsK"></iframe>
