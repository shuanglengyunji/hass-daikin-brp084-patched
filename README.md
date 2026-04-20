# Daikin AC — BRP084 patched

Home Assistant custom component for Daikin AC units running the **BRP084** DSIOT API
(firmware 2.8.0+ / 3.x, typically FTXM-R / FTXM-W / FTXA-R models).

This is HA core's stock `daikin` integration pinned to a **patched fork of
`pydaikin`** that fixes several BRP084-specific bugs.

## Fixes in this build

1. **`HEAT` mode works from HA.** The stock pydaikin BRP084 driver silently
   drops the mode write when HA sends `"hot"` (HA's legacy wire string for
   `HVACMode.HEAT`), because `REVERSE_MODE_MAP` only knows `"heat"`. Result:
   switching to Heat from the HA climate card did nothing. This build accepts
   the `"hot"` alias and sends the mode write correctly. Upstream: [pydaikin#81].
2. **Compressor frequency sensor.** Reads compressor Hz (and a run flag) from
   outdoor-unit entity `e_2006/p_04` (u16 little-endian, raw Hz). The base-class
   `compressor_frequency` property now populates, so HA's
   `sensor.<name>_compressor_frequency` entity appears.
3. **Daily energy sensor.** BRP084 reports only the aggregate `datas` array,
   not the BRP069-style cool/heat split. `today_energy_consumption` now falls
   back to the daily total instead of returning `0`, so
   `sensor.<name>_today_energy_consumption` shows actual kWh.

[pydaikin#81]: https://github.com/fredrike/pydaikin/issues/81

## Install

### Via HACS (recommended)

1. HACS → Integrations → three-dot menu → **Custom repositories**
2. Repository: `https://github.com/shuanglengyunji/hass-daikin-brp084-patched`
   Type: **Integration**
3. Install "Daikin AC (BRP084 patched)", then **restart Home Assistant**.

### Manual

```bash
# On your HA host (SSH / Samba / whatever you use)
cd /config
git clone https://github.com/shuanglengyunji/hass-daikin-brp084-patched.git /tmp/daikin-patched
mkdir -p custom_components
cp -r /tmp/daikin-patched/custom_components/daikin custom_components/
# Restart HA
```

On first start after install, HA pip-installs the patched `pydaikin` from the
fork (requires `git` on the host — HA OS ships with it). Expect 30–60s of extra
startup time on the RPi.

## Compatibility

- Tested on **Daikin FTXM71WVMA** (firmware 3.12.3).
- Expected to work on any BRP084 unit that exposes outdoor entity `e_2006`
  (most FTXM-R / FTXM-W / FTXA-R series). On units that don't expose `e_2006`,
  the compressor sensor simply won't appear — other fixes still apply.
- Requires Home Assistant 2024.1 or later.

## Rolling back

```bash
rm -rf /config/custom_components/daikin
# Restart HA — falls back to core integration with stock pydaikin
```

## Upstream

Pydaikin fixes live on [fredrike/pydaikin#TBD]. Once merged and released, this
custom component becomes obsolete — uninstall and use core.

## License

Same as pydaikin / Home Assistant core (Apache-2.0 / MIT respectively).
