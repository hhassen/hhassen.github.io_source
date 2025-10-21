---
title: "My Bitcoins almost disappeared forever: Fixing a Crashing Blockstream Green Lightning Wallet"
date: 2025-09-11T10:00:00Z
draft: false
tags: ["bitcoin", "android", "debugging"]
description: "A step-by-step walkthrough of how I rescued my Bitcoin Lightning funds from a crashing Blockstream Green app using backups, ADB logs, and a version downgrade."
---

I recently faced every Bitcoiner’s nightmare: I had just bought my first Bitcoin, set up Blockstream Green with a Lightning account, and suddenly the app on my Android phone started crashing immediately at startup. My Lightning funds were locked inside my phone, and unlike with on-chain Bitcoin, writing down my seed phrase wasn’t enough — Lightning state lives on-device.  

I contacted Blockstream support right away. Unfortunately, the process was long, and their answers didn’t solve anything:  
- They suggested updating the app — but I was already on the latest version.  
- They suggested making a backup — but for Lightning, a seed alone won’t restore your state.  

That’s when I realized: if I wanted to save my coins, I needed to find a solution myself.  

---

## Step 1: Back up everything before testing  
Before experimenting, I made sure to back up the app data. This was critical, because my Lightning channels lived inside my phone.  

```bash
adb backup -apk -obb -shared com.greenaddress.greenbits_android_wallet
```

This backup contained a SQLite database with encrypted_data, pin_data, wallet_id, and network. It confirmed that both on-chain (electrum-mainnet) and Lightning (greenlight-mainnet) information was stored locally.

---
## Step 2: Attempting decryption (and knowing when to stop)

I explored whether I could decrypt the stored data using my PIN and the app’s own encryption scheme:

🔐 **Encryption Algorithm Overview**

* Key derivation: PBKDF2 with HMAC-SHA512
* Inputs: PIN + salt (base64-decoded) + 100,000 iterations
* Output: 32-byte AES-256 key
* Decryption: AES-256 in CBC mode; IV = first 16 bytes of `encrypted_data`

I tried writing a Python script to decrypt this, but after a few attempts I decided not to push further. Decrypting wasn’t the real goal — restoring a working app environment was.

---
## Step 3: Investigating crash logs with ADB

Next, I turned to logs.

Find the package name:

```bash
adb shell pm list packages | grep green
com.blockstream.green
```


Find the running process ID:

```bash
adb shell pidof com.blockstream.green
12345
```

Dump logs:

```bash
adb logcat --pid=12345 > green_crash_log.txt
```

The logs revealed the issue:

```bash
java.lang.IllegalArgumentException: Invalid notification (no valid small icon)
```

Translation: the app was trying to start a foreground service (likely for Lightning operations) but failed because the required **notification icon** was missing. This was a coding bug — not data loss.

---
## Step 4: Reinstall attempts and support dead ends

Support had advised me to update, but reinstalling the latest APK didn’t help. For example:

```bash
adb install -r blockstream_green.apk
adb.exe: failed to install .\BlockstreamGreen-v4.1.8-productionGoogle-release.apk: Failure [INSTALL_FAILED_USER_RESTRICTED: Install canceled by user]
```

After enabling **Allow USB installs**, the reinstall worked:

```bash
adb.exe install -r .\BlockstreamGreen-v4.1.8-productionGoogle-release.apk
Success
```

But the crash persisted.

At this point, I went beyond what support had suggested: try a downgrade.

---
## Step 5: Downgrade as the real fix

Attempting to install v4.1.5 at first failed:

```bash
adb.exe install -r .\BlockstreamGreen-v4.1.5-productionGoogle-release.apk
adb.exe: failed to install .\BlockstreamGreen-v4.1.5-productionGoogle-release.apk: Failure [INSTALL_FAILED_VERSION_DOWNGRADE]
```

Solution: add the `-d` flag to force a downgrade.

```bash
adb.exe install -r -d .\BlockstreamGreen-v4.1.5-productionGoogle-release.apk
Performing Streamed Install
Success
```

And just like that: the older version opened successfully, my wallet loaded, and my Lightning funds were safe.

---
## Conclusion: what I learned

* **Lightning is device-dependent.** A seed phrase isn’t enough. Backups of the app state are critical.

* **Support isn’t always enough.** The official team couldn’t resolve this issue — updating and backing up weren’t real fixes.

* **Logs point to answers.** ADB logcat gave me the exact reason for the crash: a missing notification icon.

* **Downgrade = lifesaver.** Reverting to v4.1.5 bypassed the bug and restored full access to my funds.

* **The relief is unforgettable.** After hours of uncertainty, finally seeing my wallet open again was the moment I knew I had saved my Bitcoin from vanishing.

---

✨ This experience taught me to always back up, to follow the data, and not to rely solely on official fixes when dealing with critical funds. Sometimes, the only way forward is persistence, patience, and a willingness to get your hands dirty with the technical details.