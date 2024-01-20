**TL;DR**: You are going to get fucked by sites detecting your devtools, the easiest bypass for this is using [a web sniffer extension](https://chrome.google.com/webstore/detail/web-sniffer/ndfgffclcpdbgghfgkmooklaendohaef?hl=en)

Many sites use some sort of debugger detection to prevent you from looking at the important requests made by the browser.

You can test the devtools detector here: https://blog.aepkill.com/demos/devtools-detector/
Code for the detector found here: https://github.com/AEPKILL/devtools-detector

# How are they detecting the tools?

One or more of the following methods are used to prevent devtools in the majority of cases (if not all):

**1.**
Calling `debugger` in an endless loop.
This is very easy to bypass. You can either right click the offending line (in chrome) and disable all debugger calls from that line or you can disable the whole debugger.

**2.**
Attaching a custom `.toString()` function to an expression and printing it with `console.log()`.
When devtools are open (even while not in console) all `console.log()` calls will be resloved and the custom `.toString()` function will be called. Functions can also be triggered by how dates, regex and functions are formatted in the console.

This lets the site know the millisecond you bring up devtools. Doing `const console = null` and other js hacks have not worked for me (the console function gets cached by the detector). 

If you can find the offending js responsible for the detection you can bypass it by redifining the function in violentmonkey, but I recommend against it since it's often hidden and obfuscated. The best way to bypass this issue is to re-compile firefox or chrome with a switch to disable the console.

**3.**
Invoking the debugger as a constructor? Looks something like this in the wild:
```js
function _0x39426c(e) {
    function t(e) {
        if ("string" == typeof e)
            return function(e) {}
            .constructor("while (true) {}").apply("counter");
        1 !== ("" + e / e).length || e % 20 == 0 ? function() {
            return !0;
        }
        .constructor("debugger").call("action") : function() {
            return !1;
        }
        .constructor("debugger").apply("stateObject"),
        t(++e);
    }
    try {
        if (e)
            return t;
        t(0);
    } catch (e) {}
}
setInterval(function() {
    _0x39426c();
}, 4e3);
```
This function can be tracked down to this [script](https://github.com/javascript-obfuscator/javascript-obfuscator/blob/6de7c41c3f10f10c618da7cd96596e5c9362a25f/src/custom-code-helpers/debug-protection/templates/debug-protection-function/DebuggerTemplate.ts)

This instantly freezes the webpage in firefox and makes it very unresponsive in chrome and does not rely on `console.log()`. You could bypass this by doing `const _0x39426c = null` in violentmonkey, but this bypass is not doable with heavily obfuscated js.

Cutting out all the unnessecary stuff the remaining function is the following:
```js
setInterval(() => {
    for (let i = 0; i < 100_00; i++) {
        _ = function() {}.constructor("debugger").call(); // also works with apply
    }
}, 1e2);
```
Basically running `.constructor("debugger").call();` as much as possible without using while(true) (that locks up everything regardless).
This is very likely a bug in the browser.

**4.**
Detecting window size. As you open developer tools your window size will change in a way that can be detected.
This is both impossible to truly circumvent and simultaneously easily sidestepped.
To bypass this what you need to do is open the devtools and click settings in top right corner and then select separate window.
If the devtools are in a separate window they cannot be detected by this technique.

**5.**
Using source maps to detect the devtools making requests when opened. See https://weizmangal.com/page-js-anti-debug-1/ for further details.

# How to bypass the detection?

If you just want to see the network log that is possible with extensions, see [Web Sniffer](https://chrome.google.com/webstore/detail/web-sniffer/ndfgffclcpdbgghfgkmooklaendohaef?hl=en)
Otherwise I have patched firefox to remove any detection.

### NEW: The patch is now live in librewolf 119.0!

1. Get librewolf at https://librewolf.net/
2. Go to `about:config`
3. Set `librewolf.console.logging_disabled` to true to disable **method 2**
4. Set `librewolf.debugger.force_detach` to true to disable **method 1** and **method 3**
5. Make devtools open in a separate window to disable **method 4**
6. Disable source maps in [developer tools settings](https://github.com/Blatzar/scraping-tutorial/assets/46196380/f0ff2f24-6b8d-419c-86ac-9f47d98db749) to disable **method 5**
7. Now you have completely undetectable devtools!

---

*Old release here (you can skip this):*

I tracked down the functions making devtools detection possible in the firefox source code and compiled a version which is undetectable by any of these tools.

**Linux build**: https://mega.nz/file/YSAESJzb#x036cCtphjj9kB-kP_EXReTTkF7L7xN8nKw6sQN7gig

**Windows build**: https://mega.nz/file/ZWAURAyA#qCrJ1BBxTLONHSTdE_boXMhvId-r0rk_kuPJWrPDiwg

**Mac build**: https://mega.nz/file/Df5CRJQS#azO61dpP0_xgR8k-MmHaU_ufBvbl8_DlYky46SNSI0s

about:config `devtools.console.bypass` disables the console which invalidates **method 2**. 

about:config `devtools.debugger.bypass` completely disables the debugger, useful to bypass **method 3**. 

If you want to compile firefox yourself with these bypasses you can, using the line changes below in the described files.

**BUILD: 101.0a1 (2022-04-19)**
`./devtools/server/actors/thread.js`
At line 390
```js
  attach(options) {
    let devtoolsBypass = Services.prefs.getBoolPref("devtools.debugger.bypass", true);
    if (devtoolsBypass)
        return;
```

`./devtools/server/actors/webconsole/listeners/console-api.js`
At line 92
```js
observe(message, topic) {
let devtoolsBypass = Services.prefs.getBoolPref("devtools.console.bypass", true);
if (!this.handler || devtoolsBypass) {
  return;
}
```
`./browser/app/profile/firefox.js`
At line 23

```js
// Bypasses
pref("devtools.console.bypass", true);
pref("devtools.debugger.bypass", true);
```

### Next up: [Why your requests fail](https://github.com/Blatzar/scraping-tutorial/blob/master/disguising_your_scraper.md)
