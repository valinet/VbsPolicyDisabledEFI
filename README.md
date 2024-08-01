# VbsPolicyDisabledEFI
EFI driver which disables Virtualization-based Security (VBS) in Microsoft Windows.

## Installation

1. Download `EfiGuardDxe.efi` and `Shell_EA4BB293-2D7F-4456-A681-1F22F42CD0BC.efi` from [Releases](https://github.com/valinet/VbsPolicyDisableEFI/releases/).

2. Open administrative command window.

3. Mount EFI partition: `mountvol S: /s`.

4. Copy `EfiGuardDxe.efi` to `S:\EFI\Boot\EfiGuardDxe.efi`: `copy /Y C:\Users\%USERNAME%\Downloads\EfiGuardDxe.efi S:\EFI\Boot\EfiGuardDxe.efi`.

5. Reboot PC and start the UEFI Shell (`Shell_EA4BB293-2D7F-4456-A681-1F22F42CD0BC.efi`) EFI application. The easiest and most universal solution for doing so is to prepare a USB drive using [ventoy](https://ventoy.net/en/index.html), place the EFI files on it and finally boot it - it will find the EFI applications and allow you to launch them.

6. In UEFI Shell, identify and switch to EFI partition. To do so, start by typing `fs0:`, then Enter, then `dir` and check whether the displayed files match the expectations for an EFI partition. If not, type `fs1:`, repeat, and so on until you find the EFI partition.

7. Install the driver: `cd EFI\Boot`, then `bcfg driver add 0 EfiGuardDxe.efi "EfiGuardDxe"`.

8. Reboot into Windows. Confirm Virtualization-based Security is disabled by looking into `msinfo32` at the bottom - it should state `Not enabled`.

## Compilation

The driver is actually a patch for [EfiGuard](https://github.com/Mattiwatti/EfiGuard) which disables all functionality but the VBS disable code (EfiGuard disables VBS besides performing a whole bunch of other things).

To compile, you need to have a working [edk2](https://github.com/tianocore/edk2) build environment. The best guide that I found for setting up the build environment is this [YouTube video](https://www.youtube.com/watch?v=jrY4oqgHV0o). I recommend installing `edk2` in `C:\edk2`, and `nasm` in `C:\NASM`.

When you test compile `MdeModulePkg`, if you encounter `error 7000: Failed to execute command` and/or `error LNK2001: unresolved external symbol memcpy`, you can fix it to build completely by applying [this patch](https://github.com/tianocore/edk2/commit/73978992d8ea87bed822439a8993894d5604e9c9.patch).

My `Conf\target.txt` settings for all builds are:

```
TARGET                = RELEASE
TARGET_ARCH           = X64
TOOL_CHAIN_TAG        = VS2019
```

Make sure to run all commands in a `x86 Native Tools Command Prompt for VS 2022` window.

For building each of the projects, simply write the desired `ACTIVE_PLATFORM` one at a time in `Conf\target.txt` and then issue the following commands: `edksetup.bat` and `build`.

```
ACTIVE_PLATFORM       = MdeModulePkg/MdeModulePkg.dsc
ACTIVE_PLATFORM       = ShellPkg/ShellPkg.dsc
ACTIVE_PLATFORM       = EfiGuardPkg/EfiGuardPkg.dsc
```

When building `EfiGuardPkg`, first make sure to clone it in the correct folder - go to `edk2` folder and issue this command:

```
git clone --recursive https://github.com/Mattiwatti/EfiGuard EfiGuardPkg
```

Then, do not forget to apply the `VbsPolicyDisabledEFI.patch` patch from this repo BEFORE issuing the `edksetup.bat` and `build` commands.

The resulting file should be called placed in `C:\edk2\Build\EfiGuard\RELEASE_VS2019\X64\EfiGuardDxe.efi`.

If you need to sign this file with your own certificate (useful when running with Secure Boot enabled), use this command:

```
signtool.exe sign /fd sha256 /ac PATH_TO_DER /f PATH_TO_PFX /p CERT_PASSWORD /tr http://sha256timestamp.ws.symantec.com/sha256/timestamp /td SHA256 /sha1 SHA1_OF_CERTIFICATE "C:\edk2\Build\EfiGuard\RELEASE_VS2019\X64\EfiGuardDxe.efi"
```

## References

https://www.basicinputoutput.com/2019/10/hello-world-quick-start-with-edk2.html
