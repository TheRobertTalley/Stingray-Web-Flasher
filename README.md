# Stingray Web Flasher

Static browser-based firmware installer for Stingray ESP32 devices using
[ESP Web Tools](https://github.com/esphome/esp-web-tools).

This repository is intentionally separate from the private PlatformIO firmware
projects. It should only contain:

- Static site files such as `index.html`
- `manifest.json` files that describe flash layouts
- Compiled `.bin` firmware artifacts
- Optional project folders under `projects/`

It should never contain firmware source code, private headers, secrets, or
development configuration from the private build repo.

## Current published projects

- Device: `AIRSOFT ENERGY INDICATOR (battery meter)`
- Version, manifest, and binaries are published from the latest release build
- Device: `STINGRAY INVENTORY SYSTEM (T-Dongle S3)`
- Version, manifest, and binaries are published from the latest inventory build

Current flash layout:

- `0x0000` -> `bootloader.bin`
- `0x8000` -> `partitions.bin`
- `0xe000` -> `boot_app0.bin`
- `0x10000` -> `firmware.bin`

Those offsets were taken from the private PlatformIO build metadata in
`.pio/build/seeed_xiao_esp32c3/idedata.json` plus the generated partition table.

## Repository layout

```text
Stingray-Web-Flasher/
|-- index.html
|-- projects/
|   |-- catalog.json
|   |-- stingray-inventory-system/
|   |   |-- manifest.json
|   |   |-- bootloader.bin
|   |   |-- partitions.bin
|   |   |-- boot_app0.bin
|   |   `-- firmware.bin
|   `-- stingray-battery-meter/
|       |-- manifest.json
|       |-- bootloader.bin
|       |-- partitions.bin
|       |-- boot_app0.bin
|       `-- firmware.bin
`-- README.md
```

## End-user workflow

1. Plug the device into USB with a data cable.
2. Open the hosted page in Chrome or Microsoft Edge.
3. Click `Install` on the correct firmware card.
4. Select the serial device when the browser prompts for it.
5. Wait for flashing to complete.

No Arduino IDE, PlatformIO, CLI tooling, or backend service is required.

## Hosting

This repo is compatible with any static host that serves files over HTTP or
HTTPS, including:

- GitHub Pages
- Netlify
- A simple local web server

Do not expect `index.html` to work when opened directly from `file://`; the
page loads the project catalog with `fetch()` and should be served from a web
server.

## Updating an existing project

1. Build the private PlatformIO firmware locally.
2. Read `.pio/build/<env>/idedata.json` in the private repo.
3. Copy only the required `.bin` files into `projects/<slug>/`.
4. Update `projects/<slug>/manifest.json` with the correct decimal offsets.
5. Update `projects/catalog.json` so the landing page exposes the new version.
6. Commit and push the flasher repo.

For PlatformIO-based ESP32 builds, the useful fields in `idedata.json` are:

- `extra.flash_images` for bootloader, partition, and boot app images
- `application_offset` for the main firmware image

## Adding another firmware project

1. Create a new folder under `projects/<new-project>/`.
2. Copy the compiled `.bin` files for that board into the new folder.
3. Add a `manifest.json` beside those binaries.
4. Append a new entry to `projects/catalog.json`.
5. Commit and deploy.

The landing page is data-driven, so additional projects only need a new folder
and a new catalog entry. The newest published entry is also highlighted on the
landing page.

## Automation

This repo can be updated by a GitHub Actions workflow in the private firmware
repository. The workflow builds the latest `main` branch firmware, copies only
the required `.bin` files, and updates the manifest plus catalog in this repo.
