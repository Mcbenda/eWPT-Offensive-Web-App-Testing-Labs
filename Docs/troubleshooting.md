
# 🛠️ Mr. Robot VM: Complete Boot & Network Fix Guide

The Mr. Robot vulnerable machine (hosted on VulnHub) is notorious for crashing instantly on modern versions of VMware and failing to grab an IP address. This happens because the machine is built on an older Linux kernel (Ubuntu 14.04) that conflicts with modern VMware drivers. 

Here is the complete step-by-step guide to bypassing the crash, applying a permanent fix, and configuring your network so you can actually start hacking.

---

## Phase 1: Intercepting the Boot Crash (GRUB)
If you just power on the VM normally, it will likely hang on a black screen displaying a `vmw_balloon` error. We need to catch the machine *before* it tries to load this broken driver.

1. **Restart the VM** using the VMware controls.
2. Immediately click inside the black virtual machine window so your keyboard is captured.
3. **Hold down the `Shift` key** (or repeatedly tap `Esc`) as it boots to force the blue-and-white Bitnami GRUB boot menu to appear.
4. Use your **Up Arrow** to highlight the very first option: `Ubuntu 14.04.2 LTS, kernel 3.13.0-55-generic`.
5. Press the **`e`** key to edit the boot parameters for this option.
6. On the next screen, use your **Down Arrow** to highlight the line that starts with the word **`kernel`**.
7. Press the **`e`** key again to edit just that line.

## Phase 2: Getting a Root Shell
You are now looking at a raw text prompt. We are going to tell the system to boot into a read-write root shell instead of trying to load the full operating system.

1. Use your backspace key to delete the words `ro quiet splash` (or `ro single`) at the very end of the line.
2. Replace it by typing the following exactly:
   `rw init=/bin/bash`
3. Press **`Enter`** to confirm the edit.
4. Press the **`b`** key to boot. 
5. The screen will flash some text and then appear to hang, often with a delayed kernel message (like `random: nonblocking pool is initialized`) printing over the screen. **Press `Enter` once** to clear the text and reveal the hidden `root@(none):/#` prompt!

## Phase 3: The Permanent Fix
Now that you have root access, you need to blacklist the broken VMware memory balloon driver so it never loads again. 

1. Type the following command to add the driver to the blacklist:
   `echo "blacklist vmw_balloon" >> /etc/modprobe.d/blacklist.conf`
2. *(Note: Because Mr. Robot is built on Ubuntu 14.04, it natively uses `eth0` for networking, so you do not need to apply the modern `net.ifnames=0` GRUB fix here!)*
3. Force the machine to restart by typing:
   `reboot -f`

Let the machine boot normally. It should sail smoothly past the crash screen and drop you at the `mrrobot login:` prompt!

---

## Phase 4: Connecting the Network
Even though the machine is running, it might not have an IP address yet. You need to ensure VMware is actually virtually "plugging in" the network cable.

1. Right-click the **Mr. Robot** VM in your VMware library and select **Settings**.
2. Click on **Network Adapter**.
3. In the top right corner of that menu, ensure **BOTH** the `Connected` and `Connect at power on` boxes are checked. *(If these are unchecked, the VM is physically disconnected from the virtual switch).*
4. Ensure the **Network Connection Type** (e.g., NAT or Host-Only) matches the exact same network type your Kali Linux VM is using. 
5. Click **OK**.
6. Restart the Mr. Robot guest one more time so it requests a fresh IP address from the DHCP server.

## Phase 5: Verifying the Connection
Switch over to your Kali Linux terminal and run an ARP scan to find your target!

bash
sudo arp-scan -l


# 🛠️ Troubleshooting VulnHub Boot & IP Issues

When importing older VulnHub machines into newer versions of VMware or VirtualBox, you will frequently run into two major issues:
1. **The Boot Hang:** The machine gets stuck on the boot screen (often showing a `vmw_balloon` error).
2. **The Missing IP:** The machine boots, but running `netdiscover` or `arp-scan` yields no IP address.

These issues happen because older Linux kernels expect the network interface to be named `eth0`, but modern hypervisors assign names like `ens33` or `enp0s3`. Here is the step-by-step guide to fixing both.

---

## Step 1: Adjust Hypervisor Network Settings
Before hacking the VM's operating system, check your hypervisor settings.

* **Open VirtualBox / VMware:** Go to the settings for the specific target VM.
* **Navigate to Network Settings:** Ensure the adapter is attached to your **Host-Only Network** (or NAT, depending on your topology).
* **Change the Adapter Type (VirtualBox):** Click Advanced, and change the Adapter Type to **Intel PRO/1000 MT Desktop**. Older Linux boxes often fail to recognize the default "PCnet-FAST III" adapter.
* **Reboot the VM:** If it gets an IP, you are done. If not, proceed to Step 2.

## Step 2: Intercept the Boot Process (GRUB)
To fix the internal network configuration or bypass boot errors, we need to boot into the machine as the root user.

* **Restart the target VM.**
* **Hold the `Shift` key** immediately as it boots to bring up the GRUB bootloader menu.
* **Press `e`** to edit the boot parameters.
* **Find the line starting with `linux`** (it usually ends with `ro quiet splash`).

## Step 3: Fix the Network Naming Scheme
We need to force the Linux kernel to use the legacy `eth0` naming convention.

* **Edit the linux line:** Navigate to the end of the line starting with `linux` and add the following text: `net.ifnames=0 biosdevname=0`
* **Boot the machine:** Press `Ctrl + X` or `F10` to boot with these new settings. 
* **Check your IP:** Once booted, run your host discovery scan again. The machine should now grab an IP via DHCP using `eth0`.

## Step 4: Fix the "vmw_balloon" Boot Hang
If your machine is completely stuck booting and displaying `vmw_balloon: failed to send start command to the host`, the VMware memory balloon driver is crashing.

* **Intercept GRUB again:** Follow Step 2 to edit the boot parameters.
* **Boot into Single User Mode:** Change the `ro` (read-only) flag on the linux line to `rw` (read-write), and append `init=/bin/bash` at the very end of the line. Press `Ctrl + X` to boot.
* **Blacklist the driver:** You will now drop into a root shell. Type the following command to prevent the driver from loading: `echo "blacklist vmw_balloon" >> /etc/modprobe.d/blacklist.conf`
* **Restart the VM:** Type `reboot -f` and let the machine start normally. The hang should be gone!
