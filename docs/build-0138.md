# Build 0138: platform UUID without corecrypto

Status: BOOT image built; see the result below once booted. Contains everything
in [build 0136](build-0136.md) and the fix from [build 0137](build-0137.md).

The UUID is mixed with plain arithmetic, because corecrypto is not registered
when the platform expert starts. `tests/test_platform_uuid.py` now fails if SHA1
or crypto returns to that path.

## Reading the log

```
B2L: IOPlatformUUID published, unit-specific
```

| Sign | Meaning |
| --- | --- |
| Log and data volume unchanged after the boot | It still dies before storage: flash 0136 back. |
| About opens, no new `0x8BADF00D` for Settings | The fix works. |
| Settings hangs and configd is not in `gethostuuid` | A second cause sits behind this one. |
