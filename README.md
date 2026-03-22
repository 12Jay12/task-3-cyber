"you are a pro penetration tester , find critical vulnerabilities"


No confirmed critical vulnerability showed up in this APK from static review. The highest-impact findings are below.

1. High: plaintext credential storage and plaintext authentication
[MainActivity.smali](/Users/jayamin/Documents/New%20project/a1_case1_decompiled/smali/com/example/mastg_test0016/MainActivity.smali#L25) writes `Username: <user> Password: <pass>` directly into `credentials.txt`, and [Login.smali](/Users/jayamin/Documents/New%20project/a1_case1_decompiled/smali/com/example/mastg_test0016/Login.smali#L40) reads that same file back and compares raw strings for login. This means any attacker who can access app data on a rooted device, compromised device, forensic image, or via another app compromise gets reusable credentials immediately. There is no hashing, no salting, and no secure keystore usage.

2. Medium: session token is weak and not actually enforced
[Login.smali](/Users/jayamin/Documents/New%20project/a1_case1_decompiled/smali/com/example/mastg_test0016/Login.smali#L337) generates the session token with `java.util.Random`, which is predictable and not suitable for authentication material. More importantly, [Profile.smali](/Users/jayamin/Documents/New%20project/a1_case1_decompiled/smali/com/example/mastg_test0016/Profile.smali#L74) never validates the token before showing the profile screen, and the only references to `sessionToken` are create/read/remove helpers. In practice, the token is decorative, not access control.

3. Low: authentication result is logged
[Login$1.smali](/Users/jayamin/Documents/New%20project/a1_case1_decompiled/smali/com/example/mastg_test0016/Login$1.smali#L76) logs whether credential validation succeeded. That is not as serious as the storage issue, but it still leaks auth state into log output on debuggable/test environments.

What I did not find:
- No exported app-specific activity/service/provider abuse path in the manifest. [AndroidManifest.xml](/Users/jayamin/Documents/New%20project/a1_case1_decompiled/AndroidManifest.xml#L5)
- `allowBackup` is already `false`, which reduces one common data-extraction path. [AndroidManifest.xml](/Users/jayamin/Documents/New%20project/a1_case1_decompiled/AndroidManifest.xml#L5)

If you want, I can do the next step and turn this into a short pentest report with:
- vulnerability title
- impact
- proof of code path
- remediation
- CVSS-style severity suggestions.
