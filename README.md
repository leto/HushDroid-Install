# HushDroid-Install

## GrapheneOS Installation 

### Fastboot

You need an updated copy of the fastboot tool and it needs to be included in your PATH environment variable. You can run fastboot --version to determine the current version. It should be at least 28.0.2. 

You  need platform tools from Google. You can either obtain these as part of the standalone SDK or Android Studio which are self-updating or via the standalone platform-tools releases. For one time usage, it's easiest to obtain the latest standalone platform-tools release, 
https://developer.android.com/studio/releases/platform-tools, and extract it and add it to your PATH in the current shell. For example:

```
unzip platform-tools_r29.0.4-linux.zip export PATH="$PWD/platform-tools:$PATH"
```

Sample output from fastboot --version afterwards:
fastboot version 29.0.4-5871666 Installed as /home/username/downloads/platform-tools/fastboot

Don't proceed with the installation process until this is set up properly in your current shell.

### Obtaining signify

To verify the download of the OS beyond the security offered by HTTPS, you need the signify tool. If you don't have a way to obtain signify from a trusted package repository, such as on Windows, skip the additional verification. This is an important step, but it only makes sense if you can chain trust from your existing OS install.
On many distributions, signify is available via a signify package in the official repositories. On Debian-based distributions like Ubuntu, the package and command name were renamed to signify-openbsd. Following Debian tradition, the signify package and command are an unmaintained mail-related tool for generating mail signatures (not cryptographic signatures) with the final 3 releases from 2003-2004 made directly by the developer via the Debian package without upstream releases. This is clearly not what you want, but it's easy to end up trying to use it instead of signify-openbsd.

### Enabling OEM unlocking

Enable the developer options menu by going to Settings ➔ About phone and pressing on the build number menu entry until developer mode is enabled.
Next, go to Settings ➔ System ➔ Advanced ➔ Developer options and toggle on the 'Enable OEM unlocking' setting. 

### Unlocking Bootloader


Boot into the bootloader interface by turning off the device, and then turning it on by holding both the Volume Down and Power buttons.
The bootloader now needs to be unlocked to allow flashing new images:
fastboot flashing unlock
The command needs to be confirmed on the device.

### Obtaining factory images

The initial install will be performed by flashing the factory images. This will replace the existing OS installation and wipe all the existing data.

Download the factory images public key (factory.pub) in order to verify the factory images.
This is the content of factory.pub:
untrusted comment: GrapheneOS factory images public key RWQZW9NItOuQYJ86EooQBxScfclrWiieJtAO9GpnfEjKbCO/3FriLGX3

Verify the factory images using the signature:

```
signify -Cqp factory.pub -x crosshatch-factory-2019.06.23.05.zip.sig && echo verified
```

### Flashing factory images

Reboot into the bootloader interface to begin the flashing procedure.
Next, extract the factory images and run the script to flash them. 
Note that the fastboot command run by the flashing script requires a fair bit of free space in a temporary directory, which defaults to /tmp:
unzip crosshatch-factory-2019.06.23.05.zip cd crosshatch-pq3a.190605.003 ./flash-all.sh
Use a different temporary directory if your /tmp doesn't have enough space available:
mkdir tmp TMPDIR="$PWD/tmp" ./flash-all.sh
Wait for the flashing process to complete and for the device to boot up using the new operating system.
You should now proceed to locking the bootloader before using the device as locking wipes the data again.
On current generation devices like the Pixel 3, Pixel 3 XL, Pixel 3a and Pixel 3a XL, you'll need to reboot from the userspace fastbootd mode to the bootloader.

### Locking the bootloader

Locking the bootloader is important as it enables full verified boot. It also prevents using fastboot to flash, format or erase partitions. Verified boot will detect modifications to any of the OS partitions (vbmeta, boot/dtbo, product, system, vendor) and it will prevent reading any modified / corrupted data. If changes are detected, error correction data is used to attempt to obtain the original data at which point it's verified again which makes verified boot robust to non-malicious corruption.
In the bootloader interface, set it to locked:
fastboot flashing lock
The command needs to be confirmed on the device since it needs to perform a factory reset.
Unlocking the bootloader again will perform a factory reset.


### Disabling OEM unlocking

OEM unlocking can be disabled again in the developer settings menu within the operating system after booting it up again.


### Verifying installation

Verified boot authenticates and validates the firmware images and OS from the hardware root of trust. Since GrapheneOS supports full verified boot, the OS images are entirely verified. However, it's possible that the computer you used to flash the OS was compromised, leading to flashing a malicious verified boot public key and images. To detect this kind of attack, you can use the Auditor app included in GrapheneOS in the Auditee mode and verify it with another Android device in the Auditor mode. The Auditor app works best once it's already paired with a device and has pinned a persistent hardware-backed key and the attestation certificate chain. However, it can still provide a bit of security for the initial verification via the attestation root. Ideally, you should also do this before connecting the device to the network, so an attacker can't proxy to another device (which stops being possible after the initial verification). Further protection against proxying the initial pairing will be provided in the future via optional support for ID attestation to include the serial number in the hardware verified information to allow checking against the one on the box / displayed in the bootloader. See the Auditor tutorial for a guide.
After the initial verification, which results in pairing, performing verification against between the same Auditor and Auditee (as long as the app data hasn't been cleared) will provide strong validation of the identity and integrity of the device. That makes it best to get the pairing done right after installation. You can also consider setting up the optional remote attestation service.


## Replacing GrapheneOS with the stock OS

Installation of the stock OS via the stock factory images is the same process described above. However, before locking, there's an additional step to fully revert the device to a clean factory state.
The GrapheneOS factory images flash a non-stock Android Verified Boot key which needs to be erased to fully revert back to a stock device state. After flashing the stock factory images and before locking the bootloader, you should erase the custom Android Verified Boot key to untrust it:

````
fastboot erase avb_custom_key
```
