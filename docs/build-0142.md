# Build 0142: network stack user client

Status: BOOT image built; see the result below once booted. Contains everything
in [build 0141](build-0141.md).

configd's InterfaceNamer opens `IONetworkStack` as soon as it matches, and goes on
to `*STACK*` only if `IOServiceOpen` succeeds. Disassembly of its callback shows
`IOIteratorNext`, then `IOServiceOpen`, then a branch on the result. 0142 adds
`IONetworkStackUserClient`, which has no methods: configd uses the connection only
to register interfaces, and this port has none.

| Sign | Meaning |
| --- | --- |
| Settings opens right after boot, no `0x8BADF00D` | The fix works. |
| Reports still stop in `__wait_for_IOKit_to_quiesce` | The next step after `*STACK*` needs reading. |
