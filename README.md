# Android PowerHAL Parameters for Kona

A reference `powerhint.json` originally tuned for the Xiaomi Mi 10T Pro (`apollo`) on the Qualcomm Kona platform (Snapdragon 865). It documents PowerHAL resource nodes and actions for CPU, GPU, memory bandwidth, L3, storage, audio, rendering, and dex2oat workloads.

This is a device-derived baseline, not a universal drop-in configuration.

## Contents

- `powerhint.json` — 33 resource nodes and 80 PowerHint actions
- `.github/workflows/validate-json.yml` — syntax validation for every push and pull request

## Platform assumptions

The source configuration assumes a Kona-style topology:

| Cluster | Representative CPU | Maximum frequency in this configuration |
|---|---:|---:|
| Little | CPU 0 | 1.80 GHz |
| Big | CPU 4 | 2.4192 GHz |
| Prime / Big Plus | CPU 7 | 2.8416 GHz |

It also references Qualcomm-specific interfaces such as KGSL, LLCC/DDR bandwidth monitors, L3 latency devfreq, schedtune, and PM QoS.

## Included hint groups

The file includes actions for common and vendor-specific hints, including:

- `INTERACTION` and `LAUNCH`
- `SUSTAINED_PERFORMANCE` and `FIXED_PERFORMANCE`
- `DEVICE_IDLE` and `DISPLAY_INACTIVE`
- `AUDIO_LAUNCH` and `AUDIO_STREAMING_LOW_LATENCY`
- `EXPENSIVE_RENDERING`, `ML_ACC`, and `VIDEO_PLAYBACK`
- staged low-power CPU limits
- thermal throttling and memory pressure
- dex2oat boot and background operation

## Porting to another device

Before adapting this file, verify all of the following on the target ROM and kernel:

1. Every path in `Nodes[].Path` exists and accepts the declared value.
2. CPU policy representatives and available frequencies match the target SoC.
3. KGSL power levels use the same ordering. On Qualcomm devices, lower `pwrlevel` values generally represent higher performance.
4. LLCC, DDR, and L3 devfreq node names and units match the target kernel.
5. The ROM uses the compatible PowerHAL/libperfmgr JSON schema and actually emits the listed hints.
6. Persistent hints have a corresponding disable/reset path in the PowerHAL implementation.
7. Thermal policy remains authoritative; do not use this configuration to bypass hardware protection.

Do not blindly copy frequency values or sysfs paths to MediaTek, Exynos, Tensor, or a different Qualcomm platform.

## Validation

Validate the JSON locally with:

```sh
python -m json.tool powerhint.json > /dev/null
```

Syntax validation only confirms that the JSON parses. It does not prove that a node exists, a value is supported, or a hint is invoked on a particular ROM.

## Deployment warning

Replacing PowerHAL configuration can cause excessive heat, battery drain, instability, boot failure, or degraded performance. Use a recoverable systemless overlay where appropriate, keep the original file, and test changes incrementally.

## License

GPL-3.0. See `LICENSE`.
