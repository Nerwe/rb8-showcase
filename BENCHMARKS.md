# RB8 benchmark summary

Results are grouped by dataset, codec setting or validation scenario. Lower size, time and memory are better. A dash indicates that the comparison was not measured.

## RGB8 corpus size

| Dataset | Photos | RB8 bytes | JPEG XL e10 bytes | RB8 size saving | Smaller with RB8 |
|---|---|---|---|---|---|
| Kodak | 24 | 9,916,902 | 9,955,353 | +0.39% | 17/24 |
| Tecnick | 50 | 77,462,158 | 78,403,142 | +1.20% | 41/50 |
| Wikipedia | 4 | 4,899,730 | 5,032,447 | +2.64% | 4/4 |
| Total | 78 | 92,278,790 | 93,390,942 | +1.19% | 62/78 |

All 78 images were checked for exact RGB8 restoration. The corpus comparison measures size. Timing was recorded on the separate four-photo panel below.

## Four-photo RGB8 performance

| Codec | Total bytes | Encode (s) | Decode (ms) |
|---|---|---|---|
| RB8 | 4,811,230 | 20.765 | 492.853 |
| JPEG XL e1 | 6,296,390 | 0.128 | 65.428 |
| JPEG XL e2 | 5,605,706 | 0.296 | 65.226 |
| JPEG XL e3 | 5,401,629 | 0.419 | 192.128 |
| JPEG XL e4 | 5,338,519 | 0.698 | 216.573 |
| JPEG XL e5 | 5,259,269 | 0.940 | 228.955 |
| JPEG XL e6 | 5,224,989 | 1.228 | 242.568 |
| JPEG XL e7 | 5,083,237 | 2.080 | 245.076 |
| JPEG XL e8 | 5,040,095 | 3.422 | 260.616 |
| JPEG XL e9 | 5,024,791 | 7.116 | 272.609 |
| JPEG XL e10 | 4,911,103 | 178.769 | 305.968 |
| JPEG XL e11 | 4,798,498 | 2039.291 | 770.828 |

Windows x64, AMD Ryzen 7 250, eight workers per codec. RB8 used its parallel photographic model and JPEG XL used libjxl 0.13.0 in lossless mode. Encode timings include CLI and file I/O. Decode timings exclude PNG output encoding. The figures are sums of per-image medians. Against e10, RB8 encoded the set in 20.765 s versus 178.769 s (8.61× the throughput), produced 2.03% fewer bytes and took 1.61× as long to decode.

## Metadata round-trip and end-to-end restore

| Codec | Total bytes | Encode sum of medians (ms) | Restore sum of medians (ms) | Max restore memory (MiB) |
|---|---|---|---|---|
| RB8 model parallel | 523,618 | 6088.1 | 149.4 | 10.01 |
| JPEG XL e10 | 531,889 | 21312.0 | 159.2 | 15.12 |

Two RGB8 PNGs carried generated EXIF, ICC and XMP. Both codecs passed pixel and metadata checks. Times include process launch, I/O and restored PNG encoding. Each file was run five times. Each codec had the faster median restore on one of the two files.

## Grayscale, 16-bit and alpha formats

| Codec | Total bytes | Encode (s) | Decode (s) | Exact sample checks |
|---|---|---|---|---|
| RB8 v10 | 126,296 | 0.206 | 0.109 | 4/4 |
| JXL e7 | 83,597 | 0.174 | 0.101 | 4/4 |
| JXL e9 | 73,011 | 0.504 | 0.098 | 4/4 |
| JXL e10 | 72,720 | 2.711 | 0.115 | 4/4 |
| JXL e11 | 64,652 | 21.270 | 0.096 | 4/4 |

Four synthetic rasters at 128 × 96 cover Gray16, RGB16, RGBA16 and RGBA8. Times are sums of individual wall-time measurements and include process startup and file I/O. This is an integration check for bit depth and transparency.

## Noise holdout size

| Added noise sigma | Inputs | RB8 bytes | JPEG XL e10 bytes | RB8 size saving |
|---|---|---|---|---|
| 0 | 3 | 207,313 | 217,884 | +4.85% |
| 4 | 3 | 444,495 | 464,719 | +4.35% |

Three 320 × 256 sources per condition.

## Large rasters and deployment checks

| Scenario | RB8 result | External codec comparison |
|---|---|---|
| Raw RGBA16 stream, 20,000 × 20,000 | 3.2 GB round-tripped with a matching SHA-256. Peak Windows process working set was 8.66 MiB encode and 6.74 MiB decode. | — |
| Gray16 PNG stream, 20,000 × 20,000 | 800 MB of raw samples round-tripped with a matching SHA-256. | — |
| Linux concurrent restore | Four simultaneous 64 MiB restores passed under a 96 MiB address-space limit per process. A simulated write failure preserved the existing destination. | — |
| C buffer API on Windows | Across three shapes of a 6 MiB RGB8 input, the optimized build reduced encode time by 9.6–27.1% and decode time by 5.1–37.8%. Archives and restored samples matched byte for byte. | Compared with earlier RB8 C buffer code. |
| Metadata import | Integration checks covered EXIF, ICC and standard XMP from PNG, JPEG and WebP through PNG export. | — |
| Archive compatibility | Reference fixtures cover format versions 1–11. | — |
| Malformed input | 20,000 deterministic decoder mutations passed in the Windows SDK check. The release workflow also runs coverage-guided decoder fuzzing with AddressSanitizer. | — |
| Verified migration | Tests exercise restart, resume, rollback and detection of modified archives. | — |

Peak process working set excludes buffers in the caller process. The 3.2 GB test uses periodic synthetic input. The Linux address-space cap is a separate metric.
