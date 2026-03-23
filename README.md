this is my apk file, give me with decompile


The APK is decompiled.

Output folder:
[\/Users\/jayamin\/Documents\/New project\/a1_case1_decompiled](/Users/jayamin/Documents/New%20project/a1_case1_decompiled)

Main files:
- Manifest: [\/Users\/jayamin\/Documents\/New project\/a1_case1_decompiled\/AndroidManifest.xml](/Users/jayamin/Documents/New%20project/a1_case1_decompiled/AndroidManifest.xml)
- Resources: [\/Users\/jayamin\/Documents\/New project\/a1_case1_decompiled\/res](/Users/jayamin/Documents/New%20project/a1_case1_decompiled/res)
- Smali code: [\/Users\/jayamin\/Documents\/New project\/a1_case1_decompiled\/smali](/Users/jayamin/Documents/New%20project/a1_case1_decompiled/smali)

This decompilation was done with `apktool`, so you have resources + manifest + Smali, not reconstructed Java source. If you want, I can next:
1. inspect the package/app entry points,
2. search for API URLs, keys, or interesting code,
3. help produce Java-like source by installing a Java decompiler if you want that too.
















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









The app generates a session token in [Login.smali](/Users/jayamin/Documents/New%20project/a1_case1_decompiled/smali/com/example/mastg_test0016/Login.smali#L337) using `java.util.Random` to build a 16-character alphanumeric string. That is inappropriate for a security-sensitive value because `java.util.Random` is predictable and not designed for secrets such as session identifiers. A secure session token should come from a cryptographically secure generator like `SecureRandom` and be validated as part of real access control. In this APK, the token is stored in `SharedPreferences` in [Login.smali](/Users/jayamin/Documents/New%20project/a1_case1_decompiled/smali/com/example/mastg_test0016/Login.smali#L414), but [Profile.smali](/Users/jayamin/Documents/New%20project/a1_case1_decompiled/smali/com/example/mastg_test0016/Profile.smali#L74) never checks it before granting access, so the token is weak and effectively unused for protection.

A realistic attacker is a rooted-device user, malware running on a compromised phone, or a forensic analyst with access to app-private storage. Such an attacker can read the stored token, inspect app files, and decompile the APK to understand the generation logic. The attack path is straightforward: obtain local access, extract the stored session token or study the predictable PRNG-based generation method, then reuse or predict session values for impersonation if the app relied on them. In this case, the design is even weaker because no token validation protects the profile screen, so session handling fails to enforce authorization at all.
