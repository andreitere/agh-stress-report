# AdGuard Home on the Beryl 7 — capacity & blocking test

Standalone report: **[`agh-stress-report.html`](agh-stress-report.html)** — download it and open in a browser (GitHub shows HTML as source, not rendered).

Run on **2026-10-20** against `gl-mt3600be` (GL.iNet GL-MT3600BE, OpenWrt 21.02-SNAPSHOT, AdGuard Home 0.107.73, 4× Cortex-A53, 480 MB RAM).

The config under test is `AdGuardHome.yaml` from `raspi_stuff`, applied with four unavoidable adaptations (bind address, DNS port 3053 instead of 53, `30d`/`90d` durations converted to hours for 0.107.73, userfilter path). Everything else — DoH upstreams, DNSSEC, 300 goroutines, ratelimit 50 per /32, filter lists, UI login — is unchanged.

## Headline numbers

| | |
|---|---|
| Sustained, cached | **4.7k q/s** answered, ~4% dropped |
| Hard ceiling | **~5.5k q/s**, then drops instead of queueing |
| Single-client cap | **50 q/s** — the configured ratelimit, per /32, silently dropped |
| Household need | ~10–50 q/s at peak → ~100× headroom |
| Through dnsmasq `:53` | 4.9k q/s at 0.4 ms p50 (dnsmasq absorbs cache hits) |
| Uncached / DoH path | saturates around 500 q/s, p95 ~1 s |
| Ad/tracker hosts blocked | **75 of 88** probed |
| False positives | **0** across 44 popular sites |
| Router reboots | 0 |
| AGH restarts during load | 0 (no OOM, no respawn) |

## Notes

- The ratelimit is the first wall any single device hits. Bulk probe scripts trip it and look like mass failures; pace any test below 50 q/s per source IP.
- `querylog.file_enabled` was left `false`: writing the query log to flash produced 161 MB in ten minutes of load.
- The 10 MB filter image was grown to 32 MB — HaGeZi Pro plus the other two lists did not leave room for updates.
- DNS-level blocking does not remove inline ads (YouTube pre-rolls, Spotify audio) because they share hostnames with the content. CTV, app SDK and telemetry traffic is where it pays off.
