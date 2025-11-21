# IOAM-GOB

Linux Kernel and iproute2 Support for the Global-Opaque Block (GOB)

## Overview

This repository contains the reference implementation of the **Global-Opaque Block (GOB)**, a standards-compliant extension to the IOAM Pre-allocated Trace Option (PTO) that enables safe, programmable manipulation of global in-band metadata using eBPF.

The code hosted here accompanies the research work proposing GOB as a minimal yet powerful enhancement to IOAM, bridging standardized in-band telemetry with programmable data-plane processing in Linux. The implementation introduces kernel-level support for GOB inside the IOAM6 subsystem together with user-space extensions to `iproute2` for configuration and management.

## Key Features

* ✅ Extension of the Linux IOAM6 subsystem with GOB support
* ✅ Dedicated eBPF hook and program type for IOAM GOB processing
* ✅ Safe helper APIs for bounded access to GOB payload
* ✅ Userspace control via extended `iproute2` commands
* ✅ Support for schema-driven programmable metadata (EIP Information Elements)
* ✅ Full compliance with RFC 9197 and RFC 9486

## Repository Structure

```
ioam-gob/
├── kernel/
│   ├── patches/
│   │   └── linux-ioam-gob.patch
│   └── README.md
├── iproute2/
│   ├── patches/
│   │   └── iproute2-ioam-gob.patch
│   └── README.md
├── ebpf-examples/
│   ├── gob_counter.bpf.c
│   └── Makefile
└── docs/
    └── architecture.pdf
```

## Components

### Linux Kernel Patch

The kernel patch modifies the IOAM6 subsystem to:

* Recognize and parse the GOB via the PTO `G` flag
* Enforce alignment and size constraints
* Register a new eBPF hook point and program type (`BPF_PROG_TYPE_IOAM6_GOB`)
* Provide secure helpers:

  * `bpf_ioam6_trace_gob_load_bytes()`
  * `bpf_ioam6_trace_gob_store_bytes()`

### iproute2 Extension

The `iproute2` patch introduces:

* `gobsize` parameter for IOAM route encapsulation
* GOB schema management commands:

  * `ip ioam gobschema add`
  * `ip ioam gobschema del`
  * `ip ioam gobschema show`
* Namespace binding for GOB schemas

## Installation

### Kernel Patch

```bash
cd kernel/patches
patch -p1 < linux-ioam-gob.patch
make -j$(nproc)
make modules_install install
reboot
```

### iproute2 Patch

```bash
cd iproute2/patches
patch -p1 < iproute2-ioam-gob.patch
make
sudo make install
```

## Usage Example

Enable IOAM with GOB payload:

```bash
ip route add 2001:db8::/64 \
    encap ioam6 mode encap tundst 2001:db8::2 trace prealloc \
    ns 123 gobsize 16 dev eth0
```

Register a GOB schema:

```bash
ip ioam gobschema add 10 object gob_counter.o section ioam6_gob_counter
```

Bind schema to namespace:

```bash
ip ioam namespace set 123 gobschema 10
```

## eBPF Development

Example eBPF programs are provided in `ebpf-examples/` demonstrating how to safely update metadata inside the GOB payload using the provided helpers. These programs run at the dedicated IOAM6 GOB hook and are executed only when a valid GOB is present in the packet.

Compared to generic XDP or TC programs, the IOAM6 GOB hook eliminates the need to manually parse IPv6 extension headers, offering structured, bounded, and verifier-friendly access to the GOB region.

## Requirements

* Linux kernel with eBPF and IOAM6 enabled
* iproute2 with Generic Netlink support
* clang + llvm for eBPF compilation
* bpftool (optional, for debugging)

## Status

This repository provides a research-grade reference implementation intended for experimentation and integration testing. It is designed to support ongoing standardization efforts and feedback from the community.

## Related Publication

If you use this code, please cite the associated paper:

> *Global-Opaque Block (GOB): Safe and Programmable In-band Processing for IOAM in Linux*

BibTeX entry will be provided upon publication.

## Contributing

Contributions, issues, and discussions are welcome. Please open an issue or submit a pull request to propose improvements or report bugs.

## License

This project is released under the GNU General Public License v2.0 (GPL-2.0), consistent with the Linux kernel licensing model.

---

Maintained by Netgroup – University of Rome Tor Vergata
