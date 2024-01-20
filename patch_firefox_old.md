## NOTE: This section is old and unncessecary since these patches are already contributed to Librewolf.

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
