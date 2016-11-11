## Changes

- Fix corrupted BOOT storage
- Freeze manifest on build

### Working?

- Uses BSP kernel: https://github.com/ayufan-pine64/linux-3.10
- Graphics
- Ethernet
- WIFI
- Sound
- HW video acceleration
- CPU/GPU frequency and cores scaling
- LCD (configure uEnv.txt)
- Bluetooth
- GPIO in Android
- Camera: UVC and 5M pixel
- Touchscreen
- IR remote
- Configure resolution, LCD and eth speed with uEnv.txt
- Support different densities for different resolutions
- YouTube 1080p support
- SuperSU (experimental, do not update su binary)

### What does not?

- Due to no support for Widevine DRM, Netflix does not work,
- HDMI CEC works only on some TV models (ex. LG TV)

