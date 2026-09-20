# FrameCheck

**Can your PC run it?** FrameCheck is a free, single-page tool where you pick your CPU, GPU and RAM, choose a game, and see estimated frame rates at 1080p, 1440p and 4K. It then finds the cheapest upgrade that reaches your target FPS.

The whole site is one static file, `index.html`. There is no build step, no backend, no database and no dependencies to install.

## Features

- **FPS estimates** for 84 games across 109 CPUs and 94 GPUs, at 1080p, 1440p and 4K, from Competitive to Ultra settings, with upscaling, ray tracing and frame generation toggles
- **Bottleneck view** showing whether the CPU or GPU limits your frame rate
- **Upgrade advisor** that finds the cheapest way to reach a target FPS, or the best result for a budget
- **Best settings**, **upgrade simulator**, **best FPS per dollar**, and **PC vs PC comparison**
- **Your PC across every game**, plus **monitor** recommendations
- **Power supply**, **temperature**, and **storage** estimators
- **Build my PC**: picks the best parts list for a budget and checks socket, memory type, PSU size and cooler
- **PC builder simulator**: pick a real case, motherboard, CPU, cooler, memory, graphics card, storage and power supply. It checks the socket, DDR3 vs DDR4 vs DDR5, board and case size, card and cooler clearance, radiator fit, power supply size, length and 12V-2x6 cable, PCIe lanes and BIOS support, offers one-click fixes, and estimates FPS, temperatures and price. Memory speed, single vs dual channel and PCIe lanes change the FPS estimate
- **Benchmark log** and **leaderboards** (stored in the visitor's own browser)
- **Ask FrameCheck**: a rule-based assistant. It is not an LLM and answers only from the site's own data
- **Saved PCs**, light and dark themes, and keyboard-friendly, reduced-motion-aware UI
- **Cookie consent banner** and built-in **privacy policy, terms of use and disclosures**

All calculations run in the browser. Nothing a visitor enters is sent to a server.

## Run it locally

Open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Deploy

Upload `index.html` to any static host (Netlify, Cloudflare Pages, GitHub Pages, Vercel, an S3 bucket, or plain web hosting). If you use AdSense, also serve an `ads.txt` file from the domain root.

## Configuration

Everything you need to edit before launch is in the `CONFIG` object near the top of the main `<script>`.

| Setting | What it does |
| --- | --- |
| `legal.operator` | Your name or company, shown in the footer, privacy policy and terms |
| `legal.email` | A mailbox you read, for privacy and legal requests |
| `legal.jurisdiction` | Whose law governs the terms, for example `the State of California, USA` |
| `legal.effective` | Effective date of the policy and terms. Update it when you change them |
| `adsense.client` | Your AdSense publisher ID (`ca-pub-...`) |
| `adsense.slots` | Optional ad unit IDs for `top`, `inline`, `side` and `bottom`. With no slot ID the box stays hidden and Auto ads places ads |
| `affiliate` | Link templates for buy buttons. `{q}` is replaced by the part name. Replace `YOURTAG-20` with your real Amazon Associates tag |
| `partners` | Cards in the "Where to buy and build" section |

Any `legal` field left empty shows as a yellow `[placeholder]` on the page so it can't be missed.

## Updating the data

The model lives between the `/*MODEL_START*/` and `/*MODEL_END*/` comments.

**CPUs** are `[name, gaming performance index, socket]`. The index is relative to a Ryzen 5 3600 = 100.

```js
['Ryzen 5 5600', 135, 'AM4']
```

**GPUs** are `[name, performance index, VRAM in GB]`. The index is relative to a GTX 1660 Super = 100.

```js
['GeForce RTX 4060', 175, 8]
```

**Games** are `[id, name, group, GPU-bound FPS at 1080p High on a GTX 1660 Super, CPU-bound FPS cap on a Ryzen 5 3600, VRAM in GB at 1080p High, minimum RAM, recommended RAM, FPS cap (0 for none)]`.

```js
['cp2077', 'Cyberpunk 2077', 'Single-player and open world', 55, 95, 5.5, 12, 16, 0]
```

If you add a game, also add its entry to the per-game tables that reference it, such as `SIZES` (install size), and `RT_COST` or `FG_GAMES` if it supports ray tracing or frame generation. Add a search alias in `GAME_ALIAS` so the assistant recognizes it.

**PC builder simulator data** sits right after the model, in the `PCB_*` tables:

- `PCB_CASES`: real cases with the board sizes they take, maximum card length and cooler height, radiator positions, included fans and power supply type. The six cases were checked against maker and retailer listings. Add more the same way, and check every number against the maker's page.
- `PCB_BOARDS`: chipset, socket, memory type (DDR3, DDR4 or DDR5), size, memory slots, top memory speed, CPU overclocking, PCIe generations and NVMe boot support. These are typical chipset values, not individual board models.
- `PCB_KITS`, `PCB_COOLERS`, `PCB_PSUS` and `PCB_SSDS`: memory kits (with XMP or EXPO profile), coolers, power supplies (with native 12V-2x6 cable) and drives.
- `MEMTAB`: how much memory speed changes processor-limited frame rates on each platform. `1.00` is the speed the CPU scores assume.
- `PCB_PRESETS`: the starter builds. Use only IDs that exist in the tables above.

Card lengths are estimated from board power and the chosen card size, so they are typical values, not exact models.

**Prices** are rough US dollar guides and are kept in a few tables: `GPU_UP`, `CPU_UP`, `PLATFORM`, `RAM_PRICE`, `PSU_PRICE`, `DDR_PRICE`, `BUILD_BOARD`, `COOLER` and `BUILD_CPUS`. Review them regularly, because they go stale quickly.

The FPS numbers come from a model with baseline figures, not from measurements. Check the baselines are ones you can stand behind, since the site states they are informed by public benchmark results.

## Privacy, consent and legal

- Google Fonts and Google AdSense load **only after the visitor allows them** in the cookie banner. Nothing from Google is requested before that.
- The consent choice is stored in `localStorage` under `fc-consent-v1` for 12 months. Bump `CONSENT_VERSION` in the script if you add a new cookie or third party, so everyone is asked again.
- Turning advertising off deletes the AdSense cookies (`__gads`, `__gpi`, `__eoi`) and reloads the page.
- The privacy policy, terms and disclosures are in `<dialog>` elements near the bottom of `index.html`. They open from the footer and at `#privacy`, `#terms` and `#disclosures`.
- The Amazon Associates statement appears automatically once a real Amazon tag is configured.

Other keys the site stores in the visitor's browser: `framecheck-v1` (calculator state and saved PCs), `framecheck-bench-v1` (benchmark log) and `fc-theme` (theme).

**Important:** the bundled legal text is a starting point, not legal advice. Have a lawyer review it before launch. If you expect visitors from the EEA, the UK or Switzerland and want to serve personalized ads there, Google requires a Google-certified consent management platform, which this built-in banner is not.

## Project structure

```
index.html   The entire site: markup, styles, data model and logic
README.md    This file
```

## Contributing

Issues and pull requests are welcome. Please keep the site as a single dependency-free file, and keep the estimates honest: it is better to change a wrong number than to add a disclaimer around it.

## License

No license has been chosen yet. Add a `LICENSE` file before accepting outside contributions or reuse. Until then, all rights are reserved by the repository owner.

## Disclaimer

FPS figures, temperatures, power draw and prices are estimates for guidance only, not measurements or guarantees. Product and game names are trademarks of their owners, and this project is not affiliated with them.
