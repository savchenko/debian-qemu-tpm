## What?

Static builds of `libtpms` v0.7.7 and `swtpm` v0.5.2 for Debian Bullseye.

#### Configuration details
- libtpms: `--prefix=/usr --enable-static --with-tpm2 --with-openssl --enable-sanitizers`.
- swtpm: `--prefix=/usr --enable-static --with-seccomp`

`/usr` prefix is neccessary so that everything works with the bundled AppArmor profile: `/etc/apparmor.d/abstractions/libvirt-qemu`.

## Why?

`apt install virt-manager` does not provide a functioning TPM2 out of the box. This repository rectifies the issue.

## How?

1. Save yourself time and download pre-compiled binaries from the [Releases](https://github.com/savchenko/debian-qemu-tpm/releases) section.
2. `make install` both.
3. Install the Python modules system-wide:
   - `./swtpm-0.5.2/src/swtpm_setup/`
   - `./swtpm-0.5.2/samples/`
4. Adjust `libvirt` AppArmor profile so that it can launch `swtpm*` binaries.

If you stumble upon "permission denied" errors - enable log level 3+ in `/etc/libvirt/libvirtd.conf` and trace where it fails.

### Testing
On the host running pre-release version of the Bullseye:
- Linux 5.10.26-1 
- virt-manager 3.2.0-3
- libvirt0 7.0.0-3

Querying TPM from inside the VM:
```
$ tpm2_pcrread | head -12
sha1:
  0 : 0xBF2BA644D7415ED4A24C50B2A795BA58DCE9FA32
  1 : 0x4FCBBCA9EC6A78AD7510516E445A7E6BEE3B020D
  2 : 0xE68C21DC48C4EA61626A5D309CD3A3518F5D9EAA
  3 : 0xB2A83B0EBF2F8374299A5B2BDFC31EA955AD7236
  4 : 0x42E626C470E1E0DF350EDFB42268A270FB8E4AF7
  5 : 0xEC36E74A3ACF557CE8CAE2DE373731C6153A7F39
  6 : 0xB2A83B0EBF2F8374299A5B2BDFC31EA955AD7236
  7 : 0x518BD167271FBB64589C61E43D8C0165861431D8
  8 : 0xCEBC47EEB27FD591D7B1D18B03754194FAA56866
  9 : 0xE31C7D6093B9903E2EA04434AE6F28972ECB4D7B
  10: 0x0000000000000000000000000000000000000000
```

### Notes
- Double-check PCRs behaviour on your particular system.
- You might want to bind PCR8 instead of PCR6 ([have a read](https://www.gnu.org/software/grub/manual/grub/html_node/Measured-Boot.html))
- Don't try to seal against >8 PCRs.
