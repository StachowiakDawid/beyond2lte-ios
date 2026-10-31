# Build 0106: Settings About, backboardd restarts and launch loops

Status: kernel built and BOOT image packaged; host tests pass. Not yet installed.
Firmware and loader are unchanged from 0105.

The 0105 boot with working touch exposed three userspace failures that had not
been reachable without input.

## Settings → General → About crashed

Settings (Preferences) carries a 75,435-byte XML and a 52,207-byte DER
entitlement slot. The metadata reader capped each slot at 64 KiB and rejected
the whole signature when either slot exceeded it, so Settings had no
entitlements at all. `csops` CS_OPS_DER_ENTITLEMENTS_BLOB then returned ENOENT,
Security turned that into a CFError, and MediaServices'
`+[MSVEntitlementUtilities _checkBooleanEntitlement:task:]` releases a returned
error twice; the About page's song count reaches it through MPMediaLibrary, and
Settings crashed in `objc_release`. A scan of all 3,291 executables in the root
found Settings to be the only binary over the limit.

- `B2L_AMFI_XML_MAX` and `B2L_AMFI_DER_MAX` are now 256 KiB.
- A process without a DER slot now gets an empty blob (stock `csops_copy_token`
  zero header) instead of ENOENT, so no caller can reach that error path.
- `tests/test-large-entitlements.c` reads Settings-sized slots and rejects
  over-limit ones; `test-lifecycle.py` checks the empty-blob answer.

## backboardd aborted and SpringBoard restarted

backboardd asserted `found != slist->end ()` in
`CA::OGL::Context::push_surface` (ogl-context.cpp:1186) three times, and each
abort terminated SpringBoard. Core Animation marks cached surfaces volatile and,
before reuse, sets them non-volatile and checks the old state through
`IOSurfaceClientSetPurgeable`, IOSurfaceRootUserClient selector 20. The B2L
provider returned kIOReturnUnsupported. Selector 20 now records the requested
state and returns the previous one; B2L surfaces are wired and never
reclaimed, so contents are never reported lost
(`b2l_surface_purgeable.h`, `tests/test_surface_purgeable.py`). Whether this
is the only trigger of the assertion needs device confirmation.

## Launch loops

- MomentsUIService died in dyld: "security policy does not allow @ path
  expansion". dyld's `amfi_check_dyld_policy_self` is AMFI policy operation 90,
  which returned ENOTSUP, so dyld used its most restrictive policy. Operation 90
  now grants `AMFI_DYLD_OUTPUT_ALLOW_AT_PATH` except to set-user/group-ID
  processes; DYLD_* variables, insertion and interposing stay disallowed.
- bluetoothd, wifip2pd and fairplayd.H2 exit for lack of hardware and were
  relaunched about 50 times each. `scripts/disable-unsupported-daemons.py`
  writes launchd's own override file
  (`/private/var/db/com.apple.xpc.launchd/disabled.plist`) in the root image;
  Apple job definitions and the launchd cache are unchanged, and `--undo`
  re-enables the jobs.
- Not addressed: the speech-synthesizer audio units (a remote NSException),
  MessagesBlastDoorService (Swift assertion), dasd and webbookmarksd (single
  exceptions). They need their own evidence.
