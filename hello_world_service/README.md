# Linux Service Creation (systemd) — Quick Guide

This README summarizes how to create a **basic Linux service from scratch** using `systemd`. It is designed as a **memory aid and checklist**.

---

## Goal

Run a program **in the background**, managed by the operating system, that can be:

* Started / stopped
* Restarted automatically
* Enabled at system boot

---

## Key Concept (Mental Model)

> **Script does the work → Service file tells systemd how to run it → systemd controls everything**

A service is **not** a terminal command. It runs in a restricted, non-interactive environment.

---

## Directory Layout (Important)

| Item              | Location               | Why                              |
| ----------------- | ---------------------- | -------------------------------- |
| Executable script | `/usr/local/bin/`      | Safe, executable by systemd      |
| Service file      | `/etc/systemd/system/` | Custom system services live here |

Avoid running services directly from `/home`.

---

## Step 1: Create the Script

Create the script:

```bash
sudo nano /usr/local/bin/hello_service.sh
```

Script content:

```bash
#!/bin/bash

while true
do
    echo "Hello World from systemd service - $(date)"
    sleep 5
done
```

---

## Step 2: Set Permissions

```bash
sudo chmod 755 /usr/local/bin/hello_service.sh
sudo chown root:root /usr/local/bin/hello_service.sh
```

Verify:

```bash
ls -l /usr/local/bin/hello_service.sh
```

Expected:

```
-rwxr-xr-x
```

---

## Step 3: Test Script Manually (Mandatory)

```bash
/bin/bash /usr/local/bin/hello_service.sh
```

You should see output every 5 seconds. Stop with `Ctrl + C`.

If this fails, **do not continue**.

---

## Step 4: Create the systemd Service File

```bash
sudo nano /etc/systemd/system/hello.service
```

Service file content:

```ini
[Unit]
Description=Hello World Demo Service
After=network.target

[Service]
Type=simple
ExecStart=/bin/bash /usr/local/bin/hello_service.sh
Restart=always
RestartSec=2

[Install]
WantedBy=multi-user.target
```

---

## Step 5: Reload systemd

```bash
sudo systemctl daemon-reload
```

This tells systemd about new or modified services.

---

## Step 6: Start the Service

```bash
sudo systemctl start hello.service
```

Check status:

```bash
systemctl status hello.service
```

Expected:

```
Active: active (running)
```

---

## Step 7: View Logs (Correct Way)

Systemd captures `stdout` and `stderr` automatically.

```bash
journalctl -u hello.service -f
```

---

## Step 8: Enable Service at Boot (Optional)

```bash
sudo systemctl enable hello.service
```

---

## Common Errors and Meanings

| Error             | Meaning                                                            |
| ----------------- | ------------------------------------------------------------------ |
| `203/EXEC`        | systemd cannot execute the program (path, permission, interpreter) |
| Starts then stops | Script exits immediately                                           |
| No output         | Check logs with `journalctl`                                       |

---

## Core Rules to Remember

* Always use **absolute paths**
* Always specify the **interpreter** in `ExecStart`
* Services do **not** inherit your shell environment
* Executables belong in `/usr/local/bin`
* Logs belong in `journalctl`

---

## Minimal Command Checklist

```bash
sudo systemctl daemon-reload
sudo systemctl start hello.service
sudo systemctl status hello.service
sudo systemctl enable hello.service
journalctl -u hello.service -f
```

---

## Next Learning Steps

* One-shot services (`Type=oneshot`)
* Running services as non-root users
* Python-based systemd services
* Debugging failed services professionally

---

**This pattern is production-correct and reusable.**

