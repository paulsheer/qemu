# QEMU — Solaris 10 sun4u device-emulation fixes

This fork adds device-emulation fixes needed for Solaris 10 (SPARC,
`sun4u`) to boot and for its IDE and SCSI drivers to bind.

## What changed

Two fixes, in `hw/ide/cmd646.c` and `hw/scsi/lsi53c895a.c`:

- **cmd646 interrupt latch** — latch the MRDMODE interrupt-status bit when
  INTRQ is asserted and stop clearing it on the falling edge. Real cmd646
  silicon latches the bit until the guest writes-1-to-clear; clearing it on
  INTRQ deassert makes Solaris's uata driver poll an already-cleared bit
  after set_features and hang.
- **LSI 53C895 identity** — report the 53C895 (device id `0x000c`,
  revision `0x02`) rather than the 895A, and preserve the read-only CTEST3
  revision bits. Solaris 10's glm driver has no entry for the 895A
  (`0x0012`) but binds to the 895 at revision 1..3.

## Status

With this tree and the matching OpenBIOS tree, the machine boots and
reaches the Solaris installer, and the root filesystem mounts.

The resulting system is **not stable**: it runs briefly and then
experiences a spurious crash, typically within an hour of boot. The crash
has not been root-caused.

## You need both repositories

This QEMU fork is one half of the change. It must be paired with the
matching OpenBIOS fork
([`paulsheer/openbios`](https://github.com/paulsheer/openbios)), which
publishes the sun4u device tree Solaris expects. Using only one of the two
will not produce a bootable system.

## Building

Check out the matching OpenBIOS fork into this tree's `roms/openbios`
submodule, then build QEMU normally, for example:

```sh
./configure --target-list=sparc64-softmmu
make -j$(nproc)
```

Boot the resulting `qemu-system-sparc64` with a `-M sun4u` guest.
