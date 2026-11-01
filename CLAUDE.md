# LEDClock

ESPHome firmware for a WS2812B 7-segment LED clock. Everything lives in `esphome/`; [README.md](README.md) covers hardware, setup and features.

## Validate

Run `esphome config esphome/<device>.yaml` after every YAML change; it needs `esphome/secrets.yaml` (copy `secrets.yaml-sample.txt`). After touching `clock-rtc-pcf8563.yaml` or the ids it extends, also validate a device file that includes the `rtc:` package. There is no other build or test step. Flashing touches real hardware: leave `esphome run` / `upload` to the user.

## Package structure

- Device files hold only `substitutions:` and `packages:`, overriding the substitution defaults in `clock-common.yaml`.
- `clock-common.yaml` has no RTC components. `clock-rtc-pcf8563.yaml` uses its `i2c_*` and `timezone` substitutions and attaches itself with `!extend` on the ids `sntp_time` and `set_datetime_ctl`. Keep those ids stable, and give any new time-setting path an id the RTC package can extend the same way.
- Every `!secret` key used in the YAML must exist in `secrets.yaml-sample.txt` with a format-valid dummy value (passwords >= 8 chars, `api_key` 32-byte base64), so the sample can stand in for `esphome config`.

## Gotchas

- **Time Zone select stores its option index in flash.** Append new zones at the end only; reordering silently changes every deployed clock's zone. The option strings are matched verbatim in the `apply_timezone` script and the `tz_default_option` substitution: edit all three together.
- **Offset sign conventions differ.** `ParsedTimezone` offsets in `apply_timezone` are POSIX seconds west of UTC (US Central = `+21600`); the "UTC Offset" number is hours east of UTC (US Central = `-6`).
- **`now().is_valid()` means a real time source** (NTP, RTC or manual set). The cold-boot flash estimate is display-only and must never reach the system clock, so HA and the brightness script keep treating time as unknown.
- **ESPHome API drift.** `min_version` is 2026.9.0; older online examples break. E.g. select entities expose `current_option()` (a `StringRef`), not `.state`, since 2026.8. Check the current ESPHome docs before using lambda APIs from memory.
- **The display component is unpinned** (`ref: main`, `refresh: 0s`), so every compile pulls upstream. A build can break with no local change; check upstream first.
