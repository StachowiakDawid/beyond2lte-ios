# Build 0136: ACPM rate requests, blend flags

Status: BOOT image built; see the result below once booted. Contains everything
in [build 0135](build-0135.md).

- The rate read logs whether each domain replied, with the raw reply and the
  queue indices before and after.
- The GPU clock request no longer depends on that read, because Android does not
  use it. The boot level falls back to 156 MHz.
- The translator reads only bit 0 of the blend flags. Inside the software
  renderer only `draw_elements` reads that byte, and only that bit; the others
  serve the Metal path. Those draws now reach the shadow comparison.

## Reading the log

```
B2L ACPM: dvfs channel len=15 elem=16 queue tx a/b rx c/d
B2L ACPM: g3d reply=1 ... kHz (...)
B2L ACPM: dvfs queue tx a/b rx c/d
B2L ACPM: first change, 156000 to 433000 kHz
B2L ACPM: change took ... ns
B2L GPU: clock ... kHz (...)
```

| Sign | Meaning |
| --- | --- |
| `reply=0` everywhere, tx rear does not move | The firmware is not reading channel 0. |
| `reply=0` everywhere, tx rear moves | It reads and does not answer this request. |
| `first change`, then the log ends | The rate change stalled: take the rollback. |
| `set ... got no reply` | ACPM is left alone for the rest of the boot. |
