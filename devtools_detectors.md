**TL;DR**: You are going to get fucked by sites detecting your devtools, the best bypass for this is using [a web sniffer extension](https://chrome.google.com/webstore/detail/web-sniffer/ndfgffclcpdbgghfgkmooklaendohaef?hl=en)

Many sites use some sort of debugger detection to prevent you from looking at the important requests made by the browser.

You can test the devtools detector here: https://blog.aepkill.com/demos/devtools-detector/
Code for the detector found here: https://github.com/AEPKILL/devtools-detector

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
Fucking up the debugger with a `while (true) {}` loop. Looks something like this in the wild:
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

I do not actually know how this works, but the loop seems gets triggered in the presence of a debugger. Either way this instantly freezes the webpage in firefox and makes it very unresponsive in chrome and does not rely on `console.log()`. You could bypass this by doing `const _0x39426c = null` in violentmonkey, but this bypass is not doable with heavily obfuscated js.

If you just want to see the network log that is possible with extensions, see [Web Sniffer](https://chrome.google.com/webstore/detail/web-sniffer/ndfgffclcpdbgghfgkmooklaendohaef?hl=en)

I tracked down the functions making devtools detection possible in the firefox source code and compiled a version which is undetectable by any of these tools.

**Linux build**: //TODO

**Windows build**: https://mega.nz/file/NHJQnZZS#K1GXxZyAGeGyzAFoTWJhBe897gMVUzaHJRh6smKDT_Y

about:config `devtools.console.bypass` disables the console which invalidates **method 2**. 

about:config `devtools.debugger.bypass` completely disables the debugger and a lot more in the devtools, this should only be used to bypass **method 3**. 

If you want to compile firefox yourself with these bypasses you can, using the line changes below in the described files. You can probably make these bypasses a lot less destructive.

**BUILD: 94.0a1 (2021-09-09)** (UPDATE: I don't these code changes are enough in the latest firefox version)
`./devtools/server/actors/thread.js`
At line 190
```js
let devtoolsBypass = Services.prefs.getBoolPref("devtools.debugger.bypass", true);
const ThreadActor = devtoolsBypass ? null : ActorClassWithSpec(threadSpec, {
  initialize(parent, global) {
```

`./devtools/server/actors/webconsole/listeners/console-api.js`
At line 82
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
