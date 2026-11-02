# Nebulaps4

A rebranded, self-hostable copy of **RAW GAME** (`raw13g.github.io`) — a PS4 userland
WebKit → kernel exploit chain plus GoldHEN payload loader. All exploit code, kernel
offset tables, patch blobs and payload binaries are byte-for-byte copies of the
original; only the branding, logo and a payload-path fix were changed.

## Files

| File | What it is |
|---|---|
| `index.html` | Landing page. Reads the console firmware from the User-Agent, checks the offline cache, then forwards to `jb.html`. |
| `jb.html` | The exploit runner screen (spinner → `DONE` / `Restart your console`). |
| `jb.js` | The full chain: `?log=1` shows the step-by-step log, `?verbose=1` verbosity, `?stop=beforedouble`, `?patch=0`, `?payload=0`. |
| `core.js`, `mem.js`, `int64.js` | Memory primitive / RPC layer (third-party, own license). |
| `ps4_offsets.js` | Per-firmware kernel + WebKit offset tables (11.00 → 13.52). |
| `rpc_worker.js` | Web worker used by the primitive. |
| `payload.bin`, `payload2.bin`, `goldhen.bin` | GoldHEN payload (identical binaries). Loaded into the kernel after the JB succeeds. |
| `patches/13xx.bin` | Kernel patch blobs (10 sites each) for 13.02 / 13.50 / 13.52. |
| `cache.appcache` | Offline cache manifest (hashes regenerated for this build). |
| `LICENSE` | Upstream MIT license — keep it, it is required when redistributing. |

## Firmware support

Natively listed as supported on the landing page (kernel RVAs *measured on hardware*):

**13.02 · 13.04 · 13.50 · 13.52**

Offset entries also exist for **11.00, 11.50, 12.00, 12.50, 13.00** — the page shows
"UNSUPPORTED FIRMWARE", but one tap / press X still runs the chain. Caveats straight
from the upstream notes:

- 11.00 / 11.50 / 13.00 — marked *proven* in the offset table.
- 12.00 / 12.50 are marked **UNTESTED on hardware** and their kernel patch blobs
  (`patches/1200.bin`, `patches/1250.bin`) do not exist upstream, so the kernel-patch
  stage aborts — the WebKit stage still runs.
- 13.00's `patches/1300.bin` is also missing upstream. I did **not** alias 13.02's
  blob onto it: 13.00 and 13.02 have different `k_idt_rsvd`/`k_sysctl_handle_int`, so
  the patch sites are at different addresses and reusing it would write to the wrong
  kernel memory. 13.00 stops at the patch gate.

Check the real result on console with `jb.html?log=1` — the log prints a
`PROOF-SUMMARY-FINAL pass=… fail=…` line and every stage's PASS/FAIL.

## Run it

Any plain static web server works, as long as the console can reach it over HTTP/HTTPS.

```bash
# on your PC, from this folder
python3 -m http.server 8080 --bind 0.0.0.0
# then on the PS4 browser:  http://<your-pc-lan-ip>:8080/
```

You must open **`index.html`** (the folder root), not `jb.html` directly — the landing
page is what primes the offline cache and picks the firmware.

GitHub Pages / GitLab Pages / Cloudflare Pages: upload the folder contents as-is and
open the repo's Pages URL.

## Changes vs. the original

1. **Branding** — `RAW GAME` → **Nebulaps4** in `index.html`, `jb.html` (title + brand
   bar) and a new 96×96 nebula "N" logo (inlined as base64 exactly like the original).
2. **Payload fallback** — upstream defaults to `payload.bin`, which is **404 in the
   original repo**; only `payload2.bin` / `goldhen.bin` exist. Firmware entries 11.00,
   11.50, 12.00, 12.50, 12.02, 12.52 hit that broken default. `payload.bin` is now
   present (copy of the GoldHEN blob), so those profiles load a payload.
3. **Cache manifest** — new revision tag + SHA-256 of every asset so consoles pick up
   the new build instead of an old cached one.

## Notes / warnings

- This is an exploitation tool. Doing this to a console can brick it, ban it, or
  corrupt its storage. Only run it on a console you own, and read the upstream
  project's warnings — nothing here adds any safety guarantee.
- GoldHEN is third-party GPL software (bundled binary, unmodified) and is **not**
  covered by the repo's MIT license. Do not relicense it.
- The appcache/`manifest` attribute is legacy; on modern browsers it is ignored and
  the landing page falls back to a direct load. That's expected.
- If you edit any asset, bump the `rev` comment in `cache.appcache` and recompute that
  file's hash line, otherwise consoles keep serving the cached old copy.
