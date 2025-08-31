# Ubuntu NVIDIA Suspend/Resume Fix – "Failed to enable link training"

## Table of Contents
- [Problem Description](#problem-description)
- [Root Cause](#root-cause)
- [Solution](#solution)
- [Steps to Apply the Fix](#steps-to-apply-the-fix)
- [Explanation of the Fix](#explanation-of-the-fix)
- [Notes & Alternatives](#notes--alternatives)
- [Contributions](#contributions)
- [Thanks](#thanks)


---

## Problem Description

On some Ubuntu systems with NVIDIA GPUs, resuming from **suspend** shows the error:

Failed to enable link training

This may result in:
- A **black screen** that forces a hard reboot  
- Or a **long delay (30–50 seconds)** before the system becomes usable again  

This is a common issue reported by many Linux users.


---

## Root Cause

The problem is related to **NVIDIA proprietary drivers** and how they interact with the system during suspend/resume.  

When the CPU enters **very deep sleep states (C-states)**, the GPU sometimes fails to re-establish **PCIe link training** upon wake-up. This leads to black screens or delayed resume.  

Linus Torvalds once famously said:  
> “F*** you, NVIDIA.”  
🎥 [Video clip](https://www.youtube.com/shorts/e0xE4YBrF5E)

*(Though to be fair, NVIDIA’s Linux support has improved significantly in recent years.)*


---

## Solution

The fix is to **limit how deep the CPU can sleep** by restricting the maximum C-state to **C4**.  
This avoids the unstable deeper C-states (C5, C6, etc.) that cause the GPU driver to fail during resume.


---

## Steps to Apply the Fix

1. Open a terminal and edit the GRUB config by using:  
   ```bash
   sudo nano /etc/default/grub

2. Find the line:
   ```bash
   GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"

3. Modify it to include the parameter:
   ```bash
   GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_idle.max_cstate=4"

4. Save and exit (Ctrl+O, Enter, Ctrl+X)

5. Update GRUB:
   ```bash
   sudo update-grub

6. Reboot your system and check Suspend activity


---

## Explanation of the Fix

Modern Intel CPUs have multiple idle power states (C-states):

C0 → active

C1–C3 → light sleep (fast wake-up)

C4 → deeper sleep with reduced voltage

C5–C10 → very deep sleep (flush caches, cut voltage, longest wake-up time)

By default, Linux allows the CPU to enter the deepest available C-state.

Unfortunately, NVIDIA drivers sometimes fail to handle GPU reinitialization when the CPU is in these very deep states.

Setting intel_idle.max_cstate=4 prevents the CPU from going beyond C4, ensuring stable resume behavior.

The trade-off:

Slightly higher idle power usage and heat

But suspend/resume works reliably


---

## Notes & Alternatives

This workaround is primarily for NVIDIA GPUs on Ubuntu but may help with similar PCIe resume issues on other hardware.

Other possible tweaks (less common):

pcie_aspm=off → disables PCIe Active State Power Management

Using the open-source Nouveau driver instead of NVIDIA proprietary drivers (not always practical for performance reasons)

Testing with the latest NVIDIA drivers or newer kernels, as support is improving


---

## Contributions

If you find additional fixes, workarounds, or improvements, feel free to open an issue or pull request.
This repo exists to help others who face the same frustrating bug.

---

## Thanks
Thanks to the Linux community (forums, Reddit, GitHub issues) where this workaround was discovered and refined.  

