# Using Frida SSL Bypass with LDPlayer

## Prerequisites

1. **LDPlayer** installed and running
2. **Frida Tools** installed on your PC
3. **ADB** (Android Debug Bridge) - usually comes with LDPlayer
4. **Facebook Lite APK** installed on LDPlayer

---

## Step 1: Install Frida Tools on Your PC

Open PowerShell or Command Prompt and run:

```bash
pip install frida-tools
```

Verify installation:
```bash
frida --version
```

---

## Step 2: Enable ADB in LDPlayer

1. Open **LDPlayer**
2. Click the **Settings** icon (gear icon on the right sidebar)
3. Go to **Other Settings** tab
4. Enable **ADB Debugging** (or **Root permission**)
5. Restart LDPlayer if prompted

---

## Step 3: Connect to LDPlayer via ADB

### Find LDPlayer ADB Port

LDPlayer uses different ports for each instance:
- **Default port**: 5555, 5556, 5557, etc.
- **Check in**: LDPlayer Settings > Other Settings > ADB Port

### Connect ADB

Open Command Prompt or PowerShell:

```bash
# Navigate to LDPlayer directory (adjust path if needed)
cd "C:\LDPlayer\LDPlayer4.0.XX"

# Or use system ADB
adb connect 127.0.0.1:5555
```

If port 5555 doesn't work, try:
```bash
adb connect 127.0.0.1:5556
adb connect 127.0.0.1:5557
```

Verify connection:
```bash
adb devices
```

You should see:
```
List of devices attached
127.0.0.1:5555    device
```

---

## Step 4: Install Frida Server on LDPlayer

### Download Frida Server

1. Check your Frida version:
```bash
frida --version
```

2. Download matching Frida server from: https://github.com/frida/frida/releases

For LDPlayer (x86/x86_64), download:
- **32-bit LDPlayer**: `frida-server-X.X.X-android-x86.xz`
- **64-bit LDPlayer**: `frida-server-X.X.X-android-x86_64.xz`

### Check LDPlayer Architecture

```bash
adb shell getprop ro.product.cpu.abi
```
- If output is `x86`: use x86 version
- If output is `x86_64`: use x86_64 version

### Extract and Push Frida Server

**Windows (PowerShell):**
```powershell
# Extract the .xz file (use 7-Zip or similar)
# Rename to frida-server

# Push to device
adb push frida-server /data/local/tmp/

# Make it executable
adb shell "chmod 755 /data/local/tmp/frida-server"
```

---

## Step 5: Start Frida Server on LDPlayer

Open a **new terminal/command prompt** and run:

```bash
adb shell

# Switch to root (LDPlayer is usually rooted)
su

# Run frida-server
/data/local/tmp/frida-server &
frida -U -f com.facebook.katana -l frida_ssl_bypass.js --no-pause

# Press Ctrl+C to return to shell, then exit
exit
exit
```

**Keep this terminal open** while using Frida.

### Verify Frida Server is Running

In another terminal:
```bash
frida-ps -U
```

You should see a list of running processes on LDPlayer.

---

## Step 6: Install Facebook Lite on LDPlayer

### Option A: Install Patched APK
```bash
adb install facebook_lite_patched.apk
```

### Option B: Install Original APK (if using Frida only)
```bash
adb install facebook_lite_original.apk
```

---

## Step 7: Run Frida SSL Bypass Script

### Method 1: Spawn Mode (Recommended)

This starts the app with Frida attached from the beginning:

```bash
# Navigate to your script directory
cd C:\Users\SYRAX\Desktop\DEC\lite_dec

# Run Frida with spawn mode
frida -U -f com.facebook.lite -l frida_ssl_bypass.js --no-pause
```

**Explanation:**
- `-U`: Use USB device (works for emulators too)
- `-f com.facebook.lite`: Spawn (launch) Facebook Lite
- `-l frida_ssl_bypass.js`: Load the script
- `--no-pause`: Don't pause on startup

### Method 2: Attach Mode

If Facebook Lite is already running:

```bash
# First, start Facebook Lite manually on LDPlayer

# Then attach Frida
frida -U "Facebook Lite" -l frida_ssl_bypass.js

# OR use package name
frida -U com.facebook.lite -l frida_ssl_bypass.js
```

### Method 3: Interactive Mode (for debugging)

```bash
frida -U -f com.facebook.lite

# Then manually load the script in Frida console
%load frida_ssl_bypass.js
```

---

## Step 8: Configure Proxy for Traffic Interception

### Install Burp Suite Certificate

1. Export Burp Suite certificate (DER format)
2. Push to LDPlayer:
```bash
adb push burp-cert.der /sdcard/
```

3. On LDPlayer:
   - Settings > Security > Install from SD Card
   - Select the certificate

### Set Proxy on LDPlayer

**Method A: System Settings**
1. Long-press your WiFi network
2. Select "Modify Network"
3. Show Advanced Options
4. Set Proxy to Manual:
   - Hostname: `10.0.2.2` (this is your host PC from LDPlayer's perspective)
   - Port: `8080` (or your Burp Suite port)

**Method B: Use ADB**
```bash
adb shell settings put global http_proxy 10.0.2.2:8080
```

To remove proxy:
```bash
adb shell settings put global http_proxy :0
```

---

## Complete Workflow Example

Here's the full process from start to finish:

### Terminal 1 (Frida Server):
```bash
adb connect 127.0.0.1:5555
adb shell
su
/data/local/tmp/frida-server &
exit
exit
```

### Terminal 2 (Run Frida Script):
```bash
cd C:\Users\SYRAX\Desktop\DEC\lite_dec

# Launch Facebook Lite with SSL bypass
frida -U -f com.facebook.lite -l frida_ssl_bypass.js --no-pause
```

### In Burp Suite:
1. Open Burp Suite
2. Go to Proxy > Options
3. Set proxy listener to `*:8080` (all interfaces)
4. Start intercepting

### On LDPlayer:
1. Set proxy to `10.0.2.2:8080`
2. Facebook Lite should now be running with Frida attached
3. All HTTPS traffic will be visible in Burp Suite!

---

## Troubleshooting

### Issue 1: "Failed to spawn: unable to find application"

**Solution:** Check the correct package name:
```bash
adb shell pm list packages | grep facebook
```

Use the exact package name, e.g., `com.facebook.lite`

### Issue 2: "Failed to attach: unable to connect to remote frida-server"

**Solution:** 
- Ensure frida-server is running: `frida-ps -U`
- Restart frida-server
- Check ADB connection: `adb devices`

### Issue 3: "Unable to find process with name 'Facebook Lite'"

**Solution:**
- Use package name instead: `frida -U com.facebook.lite -l frida_ssl_bypass.js`
- Or use PID: 
```bash
frida-ps -U  # Find the PID
frida -U -p <PID> -l frida_ssl_bypass.js
```

### Issue 4: Script loads but SSL bypass doesn't work

**Solution:**
- Check Frida output for errors
- Verify the app is actually using the hooked classes
- Try the simplified script: `frida_ssl_bypass_simple.js`
- Make sure proxy certificate is installed

### Issue 5: Cannot connect to 10.0.2.2

**Solution:**
LDPlayer might use different networking. Try:
- `10.0.2.2` (standard Android emulator)
- Your actual PC IP address (e.g., `192.168.1.100`)
- Check your PC's IP: `ipconfig` (Windows) or `ifconfig` (Linux)

---

## Advanced: Automated Script

Create a batch file `run_frida.bat`:

```batch
@echo off
echo Starting Frida SSL Bypass for Facebook Lite on LDPlayer...

REM Kill existing frida-server
adb shell "su -c 'killall frida-server'"

REM Start frida-server
start /B adb shell "su -c '/data/local/tmp/frida-server &'"

REM Wait for frida-server to start
timeout /t 3

REM Set proxy
adb shell settings put global http_proxy 10.0.2.2:8080

REM Run Frida script
cd C:\Users\SYRAX\Desktop\DEC\lite_dec
frida -U -f com.facebook.lite -l frida_ssl_bypass.js --no-pause

pause
```

Double-click this file to run everything automatically!

---

## Quick Reference Commands

```bash
# Connect to LDPlayer
adb connect 127.0.0.1:5555

# Start Frida Server
adb shell "su -c '/data/local/tmp/frida-server &'"

# List processes
frida-ps -U

# Run SSL bypass (spawn mode)
frida -U -f com.facebook.lite -l frida_ssl_bypass.js --no-pause

# Run SSL bypass (attach mode)
frida -U com.facebook.lite -l frida_ssl_bypass.js

# Set proxy
adb shell settings put global http_proxy 10.0.2.2:8080

# Clear proxy
adb shell settings put global http_proxy :0

# Install APK
adb install facebook_lite_patched.apk

# Uninstall app
adb uninstall com.facebook.lite

# Check logs
adb logcat | grep -i frida
```

---

## Summary

1. ✅ Install Frida tools on PC
2. ✅ Enable ADB on LDPlayer
3. ✅ Connect ADB to LDPlayer
4. ✅ Install Frida server on LDPlayer
5. ✅ Run Frida server
6. ✅ Install Facebook Lite (patched or original)
7. ✅ Run Frida script with `frida -U -f com.facebook.lite -l frida_ssl_bypass.js --no-pause`
8. ✅ Configure proxy to intercept traffic
9. ✅ Open Burp Suite and start intercepting!

**Now you can intercept and analyze all Facebook Lite HTTPS traffic!** 🎉

