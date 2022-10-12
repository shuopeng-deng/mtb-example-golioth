# Golioth Example App

This is an example Golioth app, based on https://github.com/Infineon/mtb-example-psoc6-mcuboot-basic.
It has been tested with the PSoC 6 WiFi BT Prototyping Kit
(CY8CPROTO-062-4343W).

This example supports all Golioth services, including Over-the-Air firmware
updates:

* LightDB: https://docs.golioth.io/cloud/services/lightdb
* LightDB Stream: https://docs.golioth.io/cloud/services/lightdb-stream
* Logging: https://docs.golioth.io/cloud/services/logging
* OTA: https://docs.golioth.io/cloud/services/ota

If you haven't created an account or set up any devices with Golioth, head
over to [golioth.io](https://golioth.io/) and click on "Get Started".
Instructions for creating new projects and devices can be found in the
[Golioth Console docs](https://docs.golioth.io/golioth-console).

This example is composed of two projects: bootloader and golioth_app.
You will need to compile and flash the projects separately.
Typically, you just need to flash the bootloader one time.

The flash layout is defined in
`bootloader_cm0p/flashmap/psoc62_overwrite_single_smif.json` and looks like this:

| Partition | Flash used | Address | Size | Comment |
| --- | --- | --- | --- | --- |
| Bootloader | Internal | 0x10000000 | 0x18000 | MCUboot, bootloader_cm0p |
| App Primary | Internal | 0x10018000 | 0x1C0000 | golioth_app |
| App Secondary | External | 0x18000000 | 0x1C0000 | golioth_app |

The MCUboot upgrade strategy used is `MCUBOOT_OVERWRITE_ONLY`.

Before compiling golioth_app, make sure to set the WiFi SSID and password as
well as your Golioth PSK-ID and PSK, in `golioth_main.h`:

```c
#define WIFI_SSID "WiFiSSID"
#define WIFI_PASSWORD "WiFiPassword"
#define GOLIOTH_PSK_ID "device@project"
#define GOLIOTH_PSK "supersecret"
```

Compile and flash the bootloader:

```
cd bootloader_cm0p
make build -j8
make program
```

Compile and flash the Golioth app:

```
cd golioth_app
make build -j8
make program
```

After compiling and running, you should see a output similar to the following:

```
[INF] MCUBoot Bootloader Started
[INF] External Memory initialized w/ SFDP.
[INF] boot_swap_type_multi: Primary image: magic=good, swap_type=0x1, copy_done=0x2, image_ok=0x2
[INF] boot_swap_type_multi: Secondary image: magic=unset, swap_type=0x1, copy_done=0x3, image_ok=0x3
[INF] Swap type: none
[INF] User Application validated successfully
[INF] Starting User Application (wait)...
[INF] Start slot Address: 0x10018400
[INF] MCUBoot Bootloader finished
[INF] Deinitializing hardware...
External Memory initialized w/ SFDP.
=========================================================
[GoliothApp] Version: 1.0.0, CPU: CM4

=========================================================
[GoliothApp] Watchdog timer started by the bootloader is now turned off to mark the successful start of Golioth app.
[GoliothApp] User LED toggles at 1000 msec interval

WLAN MAC Address : C4:AC:59:77:07:20
WLAN Firmware    : wl0: Jul 18 2021 19:15:39 version 7.45.98.120 (56df937 CY) FWID 01-69db62cf
WLAN CLM         : API: 12.2 Data: 9.10.39 Compiler: 1.29.4 ClmImport: 1.36.3 Creation: 2021-07-18 19:03:20
WHD VERSION      : v2.4.0 : v2.4.0 : GCC 10.3 : 2022-08-04 17:12:02 +0800
Wi-Fi Connection Manager initialized.
Successfully connected to Wi-Fi network 'WiFiSSID'.
IP Address Assigned: 192.168.86.250
Secure Sockets initialized
I (5398) golioth_main: Waiting to Golioth to connect...
I (5430) golioth_coap_client: Start CoAP session with host: coaps://coap.golioth.io
I (5435) libcoap: Setting PSK key
I (27481) golioth_coap_client: Entering CoAP I/O loop
I (27755) golioth_main: Golioth client connected
I (27870) golioth_fw_update: Current firmware version: 1.0.0
I (27938) golioth_fw_update: Waiting to receive OTA manifest
I (28151) golioth_fw_update: Received OTA manifest
I (28151) golioth_fw_update: Manifest does not contain different firmware version. Nothing to do.
I (28153) golioth_fw_update: Waiting to receive OTA manifest
I (28225) golioth_main: Synchronously got my_int = 42
I (28226) golioth_main: Entering endless loop
I (28300) golioth_main: Callback got my_int = 42
I (28567) golioth_main: Setting loop delay to 3 s
```

If you want to test OTA firmware updates:

1. Change `APP_VERSION_MAJOR` to `2` in `golioth_app/Makefile`
2. `make build -j8`
3. Create a new artifact in Golioth: https://console.golioth.io/artifacts.
   Upload the file `golioth_app.bin` in the `build` folder, and assign it
   version `2.0.0`.
4. Create a new release in Golioth: https://console.golioth.io/releases.
   Select the artifact created in the step above.
5. Roll out the release by toggling the slider on the releases page: https://console.golioth.io/releases
6. The device (currently running version 1.0.0 firmware) will see the new
   release and start the OTA update process. If all goes well, on the next
   boot, you'll see that it has booted `[GoliothApp] Version: 2.0.0`.

This example is based on the `mtb-example-psoc6-mcuboot-basic` example from
Infineon. More details about the example are available in the README of
that project:

https://github.com/Infineon/mtb-example-psoc6-mcuboot-basic
