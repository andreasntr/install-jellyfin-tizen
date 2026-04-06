# Install Jellyfin for Samsung TV

This project makes it easy to install [Jellyfin for Samsung TV](https://github.com/jellyfin/jellyfin-tizen) with a single pre-configured docker container.

Jellyfin has yet to make it onto the Samsung App store, [but active effort is under way to get it there](https://github.com/jellyfin/jellyfin-tizen/issues/222#issuecomment-3459574549).

## Configure Computer (PC, Laptop, etc...)

- Install [Docker](https://www.docker.com/get-started/)
  * Enable any necessary [Virtualization](https://support.microsoft.com/en-us/windows/enable-virtualization-on-windows-11-pcs-c5578302-6e43-4b4b-a449-8ced115f58e1) features.

- Ensure you are connected to the same network as the TV.

## Prerequisites

### 1. Prepare Your Computer
- [Install Docker](https://www.docker.com/get-started)
- If necessary, [enable virtualization](https://support.microsoft.com/en-us/windows/enable-virtualization-on-windows-11-pcs-c5578302-6e43-4b4b-a449-8ced115f58e1)
- Ensure that your computer is on the same network as your Samsung TV

### 2. Configure Your Samsung TV

#### a) Enable Developer Mode

> [!NOTE]
> If your TV uses a right-to-left language (e.g., Arabic or Hebrew), the IP address must be entered in reverse order. [More info here.](https://github.com/Georift/install-jellyfin-tizen/issues/30)
- On the TV, open the "Smart Hub".
- Select the "Apps" panel.
- Press the "123" button (or if your remote doesn't have this button, long press the Home button) before typing "12345" with the on-screen keyboard.
- Toggle the `Developer` button to `On`.
- Enter the `Host PC IP` address of the computer you're running this container on. [Need help?](docs/troubleshooting.md)

1. On your TV, go to "Apps".
2. Scroll down to "App Settings" and open it.
3. Enter "12345" using the virtual keyboard.
4. Enable Developer Mode (**On**).
5. Enter your computer's IP address.

If you experience issues entering the IP address:
- Use an external Bluetooth keyboard.
- Use the Samsung SmartThings app to enter the IP via the virtual keyboard.

#### b) Uninstall Previous Jellyfin Versions
If Jellyfin is already installed, [follow the official uninstallation guide](https://www.samsung.com/in/support/tv-audio-video/how-to-uninstall-an-app-on-samsung-smart-tv/).

- Exact instructions will vary with the model of TV. In general you can find the TV's IP address in Settings under Networking or About. Plenty of guides are available with a quick search, however for brevity a short guide with pictures can be found [here](https://www.techsolutions.support.com/how-to/how-to-check-connection-on-samsung-smart-tv-10925).

---

## Install Jellyfin

### Receive the Device-ID
Use the following command to receive the device id from the tv

> [!NOTE]
> This command returns for some samsung tv's the model number, instead of the unique device id. If you encounter problems while installing, try to receive the unique device id via Settings->All Settings->Support->Info

```bash
docker run --rm ghcr.io/georift/install-jellyfin-tizen --ip <TV_IP> --get-device-id
```

### Installation Command
Use the following command to install Jellyfin. Replace the placeholders accordingly:

```bash
docker run --rm ghcr.io/georift/install-jellyfin-tizen --ip <TV_IP> [--build <BUILD_OPTION>] [--tag <TAG_URL>] [--oneui8 --device-id <DEVICE_ID> --email <EMAIL>] [--cert-password <PASSWORD>]
```

### Arguments
- `--ip <TV_IP>` → IP address of the Samsung TV (**required**)
- `--build <BUILD_OPTION>` → Choose a build option (default: `Jellyfin` available: `Jellyfin, Jellyfin-TrueHD, Jellyfin-master, Jellyfin-master-TrueHD, Jellyfin-secondary`)
- `--tag <TAG_URL>` → URL of the release tag (optional, uses the latest version if not specified)
- `--oneui8` → Enables One UI 8 mode (see section below)
- `--device-id <DEVICE_ID>` → Device ID (required for `--oneui8`)
- `--email <EMAIL>` → Email address (required for `--oneui8`)
- `--cert-password <PASSWORD>` → Certificate password (optional)

### Examples

#### Standard Installation with the Latest Version:
```bash
docker run --rm ghcr.io/georift/install-jellyfin-tizen --ip 192.168.0.10
```

#### Install a Specific Version with TrueHD Support:
```bash
docker run --rm ghcr.io/georift/install-jellyfin-tizen --ip 192.168.0.10 --build Jellyfin-TrueHD --tag "https://github.com/jeppevinkel/jellyfin-tizen-builds/releases/tag/2024-05-13-0139"
```

#### Install with a Custom Certificate:
```bash
docker run --rm -v "$(pwd)/author.p12":/certificates/author.p12 -v "$(pwd)/distributor.p12":/certificates/distributor.p12 ghcr.io/georift/install-jellyfin-tizen --ip 192.168.0.10 --cert-password 'CertPassw0rd!'
```

### Installation for One UI 8
If your TV uses One UI 8, follow these steps:

```bash
docker run -p 4794:4794 ghcr.io/georift/install-jellyfin-tizen --ip 192.168.0.10 --oneui8 --device-id GU43CU7179UXZG --email none@none.com
```

0. The starting process of the certificate generation script can take a while, so be patient.
1. Open `localhost:4794/auth/start` in your browser.
2. Follow the instructions on the webpage.
3. Wait for the installation to complete.

---

## Verify Installation

### Common Errors & Solutions

- **Error:** `library initialization failed - unable to allocate file descriptor table - out of memory`
  - **Solution:** Add `--ulimit nofile=1024:65536`:
    ```bash
    docker run --ulimit nofile=1024:65536 --rm ghcr.io/georift/install-jellyfin-tizen --ip 192.168.0.10
    ```

- **Error:** `install failed[118, -11], reason: Author certificate not match`
  - **Solution:** Uninstall the Jellyfin app and try again.

- **Error:** `install failed[118, -12], reason: Check certificate error: Invalid certificate chain`
  - **Solution:** Use a custom certificate as described [here](https://developer.samsung.com/smarttv/develop/getting-started/setting-up-sdk/creating-certificates.html).

### Successful Installation
If the installation is successful, you will see this message:
```txt
Installed the package: Id(AprZAARz4r.Jellyfin)
Tizen application is successfully installed.
Total time: 00:00:12.205
```

At this point you can find Jellyfin on your TV by navigating to Apps -> Downloaded (scroll down), there you'll find Jellyfin.

## Supported Platforms

---

## Supported Platforms

Currently, **amd64** is supported. ARM devices (such as Raspberry Pi or Mac M1/M2) require additional configuration.

### Installation on ARM (Mac M-Chips)
1. Enable the "Virtualization Framework".
2. Ensure that Docker supports `qemu`:
   ```bash
   docker run --rm --platform linux/amd64 alpine uname -m
   ```
   If `x86_64` is returned, everything is set up correctly.
3. Use `--platform linux/amd64`:
   ```bash
   docker run --rm --platform linux/amd64 ghcr.io/georift/install-jellyfin-tizen --ip 192.168.0.10
   ```

---

## Acknowledgments
This project would not be possible without the following:

- [jellyfin-tizen](https://github.com/jellyfin/jellyfin-tizen)
- [jeppevinkel/jellyfin-tizen-builds](https://github.com/jeppevinkel/jellyfin-tizen-builds) for providing development builds
- [vitalets/docker-tizen-webos-sdk](https://github.com/vitalets/docker-tizen-webos-sdk) for a docker container preinstalled with the Tizen SDK

## Similar Projects

Here are some similar projects we've been told of, links are provided here without
any endorsement for their quality.

- [PatrickSt1991/Samsung-Jellyfin-Installer](https://github.com/PatrickSt1991/Samsung-Jellyfin-Installer):
  * GUI based installed rather than Docker

Feel free to raise a PR with any additional projects.
