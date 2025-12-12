---
layout: project
title: "MacOS Unified Logging: Sysmon-Style Security Telemetry to Splunk"
date: 2025-12-06
categories: [projects]
tags: [macos, logging, splunk, unified-logs, security-telemetry]
description: >
  Collecting macOS security telemetry from the Apple unified logging system
  using logd predicates and Splunk Universal Forwarder, with private data
  enabled via MDM configuration profiles.
image:
  path: /assets/img/projects/mac_os.jpg
---

## MacOS Unified Logging: Sysmon-Style Security Telemetry to Splunk

## Project Overview

The goal of this project is to build a **Sysmon-style security telemetry pipeline for macOS** using **Apple’s Unified Logging system**, a **Splunk Universal Forwarder (UF)**, and a **custom Splunk add-on**.

On Windows, many blue teams rely on **Sysmon** to capture rich events like:

- Process creation and termination  
- Network connections  
- Logon/logoff activity  
- File and registry changes  

On macOS, there is **no Sysmon**, but we can get **similar visibility** by:

1. Enabling **private data** in Apple’s unified logs via an **MDM configuration profile**.
2. Using **`log` / `logd` predicate filters** to select interesting security events.
3. Streaming those events to Splunk via a **Splunk UF scripted input**.

> The result is a reusable, “Sysmon-inspired” logging baseline for macOS endpoints.

---

## Architecture

High-level components:

1. **macOS Endpoint**
   - Apple unified logging system (`logd`) keeps logs in binary store (`/var/db/diagnostics/`).
   - A **configuration profile** enables “private data” fields so user and process identifiers are not redacted.
   - Splunk Universal Forwarder runs as a daemon.

2. **Splunk UF + Custom Add-on**
   - A **scripted input** runs `log stream` (or `log show`) with **predicate filters** to pull only security-relevant events.
   - Output is formatted as **JSON** to simplify field extraction.
   - Data is sent to Splunk indexers with dedicated **sourcetypes** (e.g. `macos:unifiedlog:security`).

3. **Splunk Indexers / Search Head**
   - Indexes macOS unified log events.
   - Dashboards and detections map to Sysmon-like use cases (process execution, persistence, logon, etc.).

---

## Prerequisites

- **Managed macOS devices** (corporate-owned, supervised where possible).
- An **MDM** (e.g. Jamf Pro) capable of deploying configuration profiles.
- **Splunk Universal Forwarder for macOS** installed on endpoints.
- A **custom Splunk TA** or local app on the UF to run scripts and define sourcetypes.

> ⚠️ **Privacy note**: Enabling private unified log data is appropriate for **corporate-owned devices** but not for personal BYOD machines. Make sure you have policy and legal sign-off before deploying broadly.

---

## Step 1 – Enable Private Data in Unified Logs via Configuration Profile

By default, macOS unified logs **redact “private data”** such as usernames, host identifiers, and certain process details, which makes them much less useful for security. Jamf outlines a way to **enable private data** at scale using a **configuration profile** with a `com.apple.system.logging` payload and `Enable-Private-Data` set to `true`.

### 1.1 Create the configuration profile

Create a profile (XML / `.mobileconfig`) with this payload structure (simplified):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>PayloadContent</key>
  <array>
    <dict>
      <key>PayloadDisplayName</key>
      <string>ManagedClient logging</string>
      <key>PayloadEnabled</key>
      <true/>
      <key>PayloadIdentifier</key>
      <string>com.apple.logging.ManagedClient.1</string>
      <key>PayloadType</key>
      <string>com.apple.system.logging</string>
      <key>PayloadUUID</key>
      <string>ED5DE307-A5FC-434F-AD88-187677F02222</string>
      <key>PayloadVersion</key>
      <integer>1</integer>
      <key>System</key>
      <dict>
        <key>Enable-Private-Data</key>
        <true/>
      </dict>
    </dict>
  </array>
  <key>PayloadDescription</key>
  <string>Enable Unified Log Private Data logging</string>
  <key>PayloadDisplayName</key>
  <string>Enable Unified Log Private Data</string>
  <key>PayloadIdentifier</key>
  <string>C510208B-AD6E-4121-A945-E397B61CACCF</string>
  <key>PayloadRemovalDisallowed</key>
  <false/>
  <key>PayloadScope</key>
  <string>System</string>
  <key>PayloadType</key>
  <string>Configuration</string>
  <key>PayloadUUID</key>
  <string>D30C25BD-E0C1-44C8-830A-964F27DAD4BA</string>
  <key>PayloadVersion</key>
  <integer>1</integer>
</dict>
</plist>
```

This profile enables the **System → Enable-Private-Data** setting so that previously redacted fields become visible in unified logs.

### 1.2 Sign the configuration profile

Apple and Jamf recommend **signing** this profile before deploying it:

```bash
# Find signing identities
/usr/bin/security find-identity -p codesigning -v

# Use one of the listed identities to sign the profile
/usr/bin/security cms -S -Z "<IDENTITY_HASH>"   -i "/path/to/unsigned/profile.mobileconfig"   -o "/path/to/signed/profile.mobileconfig"
```

Some MDM workflows may otherwise alter the profile in a way that breaks the private data setting.

### 1.3 Deploy via Jamf (or your MDM)

- Upload the **signed** profile into Jamf Pro (or equivalent).
- Scope it to your **test Macs** first.
- Confirm the profile is installed on the endpoint.

### 1.4 Validate private data is visible

On a test Mac:

```bash
# Without profile, many fields show "<private>"
# After profile install, you should see usernames, etc.
log show --last 5m --info --predicate 'eventMessage CONTAINS "AuthenticationAllowed"'
```

You should now see **de-obfuscated user records** instead of `<private>` for relevant events.

---

## Step 2 – Splunk UF Add-on Design

We’ll use a **scripted input** on the UF that:

- Runs `log stream` with **predicate filters**.
- Uses `--style json` for parseable output.
- Ships events to Splunk on a a dedicated sourcetype.

### 2.1 Example Splunk script

Example shell script:  
`/opt/splunkforwarder/etc/apps/TA-macos-unifiedlog/bin/macos_unifiedlog_sysmon.sh`:

```bash
#!/bin/zsh

# Stream unified logs with predicates focused on security-relevant activity.
# Adjust --last / --predicate / --info to tune volume and coverage.

exec /usr/bin/log stream   --style json   --info   --predicate '
    (
      -- AUTH / LOGON / SCREEN LOCK
      (eventMessage CONTAINS[c] "LWScreenLockAuthentication")
      OR (process == "screensharingd" AND eventMessage CONTAINS "Authentication:")
      OR (subsystem == "com.apple.Authorization")
    )
    OR
    (
      -- PROCESS EXECUTION / GATEKEEPER / PERSISTENCE
      (process == "syspolicyd" AND subsystem == "com.apple.syspolicy.exec")
      OR (subsystem == "com.apple.loginwindow.logging"
          AND eventMessage CONTAINS "performAutolaunch")
      OR (eventMessage CONTAINS[c] "execve"
          OR eventMessage CONTAINS[c] "posix_spawn")
    )
    OR
    (
      -- NETWORK / REMOTE ACCESS
      (process == "socketfilterfw")
      OR (process == "pfctl")
      OR (process == "screensharingd" AND eventMessage CONTAINS "Authentication:")
    )
    OR
    (
      -- TCC / PRIVACY PERMISSIONS CHANGES
      (eventMessage CONTAINS[c] "Update Access Record:")
    )
  '
```

> Note: This is a **starting point**. You’ll want to test on real endpoints and tune predicates (and maybe split this into separate scripts/sourcetypes for clarity).

### 2.2 Splunk `inputs.conf`

On the UF, in the same app:

```ini
[script://./bin/macos_unifiedlog_sysmon.sh]
disabled = 0
interval = 0          ; 0 = run as a long-lived stream
sourcetype = macos:unifiedlog:security
index = security_macos
splunkd = true
```

- `interval = 0` tells the UF to treat it as a **streaming command** (like `tail -f`).
- If you’d rather run periodic snapshots, you can switch to `log show --last 1m` and set `interval = 60`.

---

## Step 3 – Sysmon-Style Predicate Filters for macOS Unified Logs

Below is a **reference list** of `log` predicate filters aimed at approximating key Sysmon events on macOS. These are meant to be used with commands like:

```bash
log stream --style json --info --predicate 'PREDICATE_HERE'
# or
log show --style json --info --last 1h --predicate 'PREDICATE_HERE'
```

You can embed these predicates into your Splunk scripts (one big OR, or separate streams per use case).

> **Important:** Unified logs do **not** perfectly mirror Sysmon, and predicates can vary across macOS versions. Use `log help predicates` and empirical testing on your own fleet to validate/adjust filters.

---

### 3.1 Logon / Logoff / Screen Lock (Sysmon-like: Security Logon/Logoff)

**Use case:** Track logon, unlock, and screen lock events with method (password, Touch ID, Apple Watch).

Predicate based on `LWScreenLockAuthentication`:

```text
eventMessage CONTAINS "LWScreenLockAuthentication"
```

More specific (logon/unlock w/ method):

```text
eventMessage CONTAINS "LWScreenLockAuthentication"
AND (
  eventMessage CONTAINS "| Verifying"
  OR eventMessage CONTAINS "| Using"
)
```

Example command:

```bash
log stream --style json --info   --predicate 'eventMessage CONTAINS "LWScreenLockAuthentication"'
```

---

### 3.2 Remote Desktop / Screen Sharing (Sysmon-like: RDP Session)

Show **Screen Sharing (VNC) authentication** events:

```text
process == "screensharingd"
AND eventMessage CONTAINS "Authentication:"
```

Example:

```bash
log stream --style json --info   --predicate 'process == "screensharingd" AND eventMessage CONTAINS "Authentication:"'
```

This approximates Sysmon events for remote desktop connections.

---

### 3.3 Process Execution / Gatekeeper Decisions (Sysmon: Process Creation, Image Load)

Gatekeeper and execution policy decisions appear under `syspolicyd` / `com.apple.syspolicy.exec`.

Predicate:

```text
process == "syspolicyd"
AND subsystem == "com.apple.syspolicy.exec"
```

Example:

```bash
log stream --style json --info   --predicate 'process == "syspolicyd" AND subsystem == "com.apple.syspolicy.exec"'
```

This captures **allow/deny decisions** about running binaries (similar to Sysmon’s process execution plus some code integrity info).

Additionally, you can look for exec/fork related markers:

```text
eventMessage CONTAINS[c] "execve"
OR eventMessage CONTAINS[c] "posix_spawn"
OR eventMessage CONTAINS[c] "exec "
```

Example:

```bash
log stream --style json --info   --predicate 'eventMessage CONTAINS[c] "execve" OR eventMessage CONTAINS[c] "posix_spawn"'
```

---

### 3.4 Persistence via Login Items (Sysmon: Autoruns)

Adding Login Items (persistence) appears under subsystem `com.apple.loginwindow.logging` with messages like `performAutolaunch`.

Predicate:

```text
subsystem == "com.apple.loginwindow.logging"
AND eventMessage CONTAINS "performAutolaunch"
```

Example:

```bash
log stream --style json --info   --predicate 'subsystem == "com.apple.loginwindow.logging" AND eventMessage CONTAINS "performAutolaunch"'
```

This approximates Sysmon’s view of autorun/persistence entries being executed at logon.

---

### 3.5 Auth Events (Sysmon: Logon, Privilege Use)

macOS **authorization daemon (`authd`)** and related authorization frameworks log via subsystem `com.apple.Authorization`.

Predicate:

```text
subsystem == "com.apple.Authorization"
```

Example:

```bash
log stream --style json --info   --predicate 'subsystem == "com.apple.Authorization"'
```

This can surface **authentication/authorization** attempts (sudo use, privilege escalations, etc.) but you’ll want to tune further based on keywords in `eventMessage`.

---

### 3.6 TCC / Privacy Permission Changes (Sysmon: Registry/Policy Changes)

Changes in macOS **Transparency, Consent, and Control (TCC)** permissions can show logs with `"Update Access Record:"` in the message.

Predicate:

```text
eventMessage CONTAINS[c] "Update Access Record:"
```

Example:

```bash
log stream --style json --info   --predicate 'eventMessage CONTAINS[c] "Update Access Record:"'
```

This roughly approximates Sysmon events for **permission and policy changes** related to camera/mic/filesystem access.

---

### 3.7 Network Security / Firewall (Sysmon: Network Connect, Firewall)

For macOS application firewall and packet filter activity, you commonly see processes like `socketfilterfw` and use of `pfctl`.

Predicates:

```text
process == "socketfilterfw"
OR process == "pfctl"
```

Example:

```bash
log stream --style json --info   --predicate 'process == "socketfilterfw" OR process == "pfctl"'
```

You can also filter further based on `eventMessage` containing `Deny`, `Allow`, certain ports, etc.

---

### 3.8 VNC / Screen Sharing Success/Failure

You might split out **success vs failure** based on `eventMessage` contents (e.g., `Authentication: SUCCEEDED` vs `FAILED`):

```text
process == "screensharingd"
AND eventMessage CONTAINS "Authentication: SUCCEEDED"
```

or

```text
process == "screensharingd"
AND eventMessage CONTAINS "Authentication: FAILED"
```

---

## Step 4 – Mapping to Splunk Sourcetypes & CIM

Once data is flowing, you can map these unified log events to Splunk’s **CIM** data models (Endpoint, Authentication, etc.), e.g.:

- `macos:unifiedlog:auth` → Authentication data model
- `macos:unifiedlog:process` → Endpoint.Processes
- `macos:unifiedlog:network` → Network_Traffic

Your scripted input can:

- Either ship **all predicates together** under a single sourcetype, or  
- Run multiple scripts with different predicates and sourcetypes, which often makes field extractions simpler.

---

## Step 5 – Tuning, Performance, and Retention

A few practical notes:

- Unified logs are **extremely verbose**; predicates are critical to avoid overwhelming endpoints and Splunk.
- Use `--last` and short time windows for initial testing, then move to `log stream`.
- macOS unified logs typically retain ~30 days of data; if you rely on **historic** investigations, consider periodic `log collect` snapshots for deep forensics alongside this live telemetry.

---

## Future Enhancements

- Incorporate **BSM audit logs** and/or **Endpoint Security (ES) API** for even richer, Sysmon-like coverage (file operations, network connections, process creation).
- Add **detections** for:
  - Suspicious persistence (new login items).
  - Unusual `syspolicyd` blocks for untrusted binaries.
  - Off-hours remote access (Screen Sharing).
- Build Splunk dashboards by:
  - User and host activity timelines.
  - Auth + process + network correlation.

---

By combining **unified log private data**, **predicate-based filtering**, and **Splunk UF scripted inputs**, this project gives your macOS fleet a **much more Sysmon-like telemetry surface**, which you can then use for hunting, alerting, and long-term investigations.
