# cl0x889

A reverse engineering challenge. Small window, one input field, one status bar.
Your goal is to find the password that turns the bottom bar green.

---

## Overview

| | |
|---|---|
| **What it is** | Reverse engineering challenge. Small window, one input field, one status bar. Your goal is to find the password that turns the bottom bar green. |
| **Password length** | Exactly 16 bytes |
| **Wrong input** | Red `LOCKED` bar, window shakes |
| **Correct password** | Green `UNLOCKED` bar |
| **3 wrong attempts** | The exe deletes itself from disk |
| **Debugger detected** | Process exits silently, window never appears |
| **Close button / Alt+F4** | Blocked. The only ways to close: correct password, three failed attempts, or Task Manager |

---

## Description

`cl0x889` is a small Windows GUI application built as a training ground for
reverse engineering. It presents a single window with a password input field
and a status bar at the bottom. The status bar is red by default. When the
correct password is entered, it turns green.

The password check is deliberately obfuscated. Strings that look like passwords
are decoys. The real password is not stored anywhere in the binary as a plain
string — it is assembled at runtime from several tables and arithmetic operations.

The application also carries a set of protections designed to make static and
dynamic analysis harder: anti-debug checks, a kill-switch that reacts to known
reverse engineering tools running on the system, and a self-destruct mechanism
that triggers after three failed attempts.

This is **not malware**. There are no network calls, no registry writes, no
persistence, no file encryption, no privilege escalation. The only "aggressive"
behavior is deleting its own executable after three failed attempts. Everything
is open source and auditable.

---

## How it looks

- **Background:** solid black.
- **Top:** a moth image (JPG, embedded as a resource).
- **Middle:** a password field with an owner-drawn grey `...` button on the right.
- **Bottom:** a wide rectangular status bar — red `LOCKED`, green `UNLOCKED`,
  or yellow `TRAP` if a specific decoy is used.
- **Top-right corner:** attempt counter `try 1/3`.
- **Window title:** `cl0x889`.
- **Window icon:** a green circuit moth, embedded in the exe.

The window is fixed-size, has no minimize-box behavior worth mentioning, and
cannot be closed with the standard X button or Alt+F4.

---

## Rules

1. Password must be **exactly 16 bytes** long.
2. Wrong input → red bar, window shakes, attempt counter increments.
3. Three wrong inputs → **self-destruct**: the exe writes a small `.bat` file
   to `%TEMP%`, launches it hidden via `cmd.exe /c`, and immediately exits.
   The bat waits a couple of seconds, then deletes the exe. After 2–3 seconds,
   the exe is gone from disk — no Recycle Bin, no warning.
4. Correct input → green bar `UNLOCKED`. The window stays open (you can close it
   manually afterward if you want — but only via Task Manager, since X and Alt+F4
   are still blocked).
5. If any of the known reverse engineering tools is running on the system, the
   process exits silently **before** the window is created.

---

## What it protects against

| Protection | Description |
|---|---|
| Anti-debug (startup) | `IsDebuggerPresent()` and `CheckRemoteDebuggerPresent()` are called before the window is shown. If either returns true, the process exits. |
| Kill-switch (process scan) | The process list is enumerated at startup. If any known RE tool is found, the process exits silently. Covered tools include x64dbg, x32dbg, IDA, Ghidra, OllyDbg, WinDbg, CDB, dnSpy, Process Hacker, Process Monitor, Frida, Scylla, Cheat Engine, ReClass.NET, Binary Ninja, radare2, Cutter, Immunity Debugger. |
| Close-button block | `SC_CLOSE` is removed from the system menu. `WM_CLOSE`, `WM_SYSCOMMAND` with `SC_CLOSE`, and `WM_SYSKEYDOWN` with `VK_F4` are all swallowed. The window icon is removed so there's no right-click menu on it. |
| Self-destruct | After three failed attempts, a `.bat` file is written to `%TEMP%`, launched hidden, and the exe is deleted in a loop with retries. The bat then deletes itself. |
| Obfuscated core | The password is not stored as a string. It is reconstructed at runtime from three tables using XOR and modular addition. Numerous decoy strings, dead branches, and unused hash/cipher functions are scattered throughout the binary to confuse disassemblers. |

---

## What it does NOT do

- No network access.
- No registry modification.
- No persistence.
- No file encryption.
- No privilege escalation.
- No interaction with other processes (only reading the process list).
- No hidden payloads.

The only side effect besides the GUI is self-deletion after three failed attempts.

---

## Files in the repository

| File | Purpose |
|---|---|
| `core.c` | Password verification core. Intentionally obfuscated. |
| `gui.cpp` | GUI implementation. WinAPI + GDI+. |
| `res.rc` | Resource script: icon, moth image, manifest. |
| `manifest.xml` | Application manifest (modern controls, DPI-aware). |
| `app.ico` | Application icon (circuit moth, green). |
| `moth.jpg` | Moth image displayed above the password field. |
| `gen.py` | Generator for the obfuscation tables. Change the password, run the script, paste the output into `core.c`. |
| `README.md` | This file. |
| `info.txt` | Short reminder for whoever finds the exe on disk. |

---

## ⚠ IMPORTANT BEFORE RUNNING

**Make a copy of `cl0x889.exe` before the first launch.** After three failed
attempts, the original will be deleted from disk — no warning, no Recycle Bin.
This is intentional and irreversible.

If you are analyzing the binary, save a backup somewhere outside the project
folder. The self-destruct logic uses a small `.bat` file in `%TEMP%` and the
`del` command. It will not work on read-only media or in directories where the
current user has no write permission — in that case the exe survives.

---

## Disclaimer

Educational project. No network, no registry, no persistence, no hidden actions.
The only "aggressive" behavior is deleting its own executable after three failed
attempts. Use at your own risk. The author is not responsible for any use
outside educational contexts.

---

## Hints

No hints. The password is in `core.c`. If you can't find it — that's the point.

> 🦋
