# Golioth Example App

This is an example Golioth app, based on https://github.com/Infineon/mtb-example-psoc6-mcuboot-basic.
It has been tested with the PSoC 6 WiFi BT Prototyping Kit
(CY8CPROTO-062-4343W).

This example requires ModusToolbox 3.0.0.

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

You will need to install python modules from MCUboot in order for
application image signing to succeed (one time only):

```
cd ../../<mtb_shared>/mcuboot/<tag>/scripts
python -m pip install -r requirements.txt
```

Now you are ready to build the bootloader and app projects.

Compile and flash the bootloader:

```
cd bootloader_cm0p
make build_proj -j8
make program_proj
```

Compile and flash the Golioth app:

```
cd golioth_app
make build_proj -j8
make program_proj
```

More detailed usage instructions, along with support for other IDEs and tools,
is available in the [Using the code example](#using-the-code-example) section.

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

1. Change `APP_VERSION_MAJOR` for the `BOOT` image to `2` in `golioth_app/Makefile`
2. `make build_proj -j8`
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

## Using the code example

Create the project and open it using one of the following:

<details><summary><b>In Eclipse IDE for ModusToolbox&trade; software</b></summary>

1. Click the **New Application** link in the **Quick Panel** (or, use **File** > **New** > **ModusToolbox&trade; Application**). This launches the [Project Creator](https://www.infineon.com/ModusToolboxProjectCreator) tool.

2. Pick a kit supported by the code example from the list shown in the **Project Creator - Choose Board Support Package (BSP)** dialog.

   When you select a supported kit, the example is reconfigured automatically to work with the kit. To work with a different supported kit later, use the [Library Manager](https://www.infineon.com/ModusToolboxLibraryManager) to choose the BSP for the supported kit. You can use the Library Manager to select or update the BSP and firmware libraries used in this application. To access the Library Manager, click the link from the **Quick Panel**.

   You can also just start the application creation process again and select a different kit.

   If you want to use the application for a kit not listed here, you may need to update the source files. If the kit does not have the required resources, the application may not work.

3. In the **Project Creator - Select Application** dialog, choose the example by enabling the checkbox (under the "WiFi" section, or just search for "golioth").

4. (Optional) Change the suggested **New Application Name**.

5. The **Application(s) Root Path** defaults to the Eclipse workspace which is usually the desired location for the application. If you want to store the application in a different location, you can change the *Application(s) Root Path* value. Applications that share libraries should be in the same root path.

6. Click **Create** to complete the application creation process.

For more details, see the [Eclipse IDE for ModusToolbox&trade; software user guide](https://www.infineon.com/MTBEclipseIDEUserGuide) (locally available at *{ModusToolbox&trade; software install directory}/docs_{version}/mt_ide_user_guide.pdf*).

</details>

<details><summary><b>In command-line interface (CLI)</b></summary>

ModusToolbox&trade; software provides the Project Creator as both a GUI tool and the command line tool, "project-creator-cli". The CLI tool can be used to create applications from a CLI terminal or from within batch files or shell scripts. This tool is available in the *{ModusToolbox&trade; software install directory}/tools_{version}/project-creator/* directory.

Use a CLI terminal to invoke the "project-creator-cli" tool. On Windows, use the command line "modus-shell" program provided in the ModusToolbox&trade; software installation instead of a standard Windows command-line application. This shell provides access to all ModusToolbox&trade; software tools. You can access it by typing `modus-shell` in the search box in the Windows menu. In Linux and macOS, you can use any terminal application.

This tool has the following arguments:

Argument | Description | Required/optional
---------|-------------|-----------
`--board-id` | Defined in the `<id>` field of the [BSP](https://github.com/Infineon?q=bsp-manifest&type=&language=&sort=) manifest | Required
`--app-id`   | Defined in the `<id>` field of the [CE](https://github.com/Infineon?q=ce-manifest&type=&language=&sort=) manifest | Required
`--target-dir`| Specify the directory in which the application is to be created if you prefer not to use the default current working directory | Optional
`--user-app-name`| Specify the name of the application if you prefer to have a name other than the example's default name | Optional

<br>

The following example clones the "[mtb-example-golioth](https://github.com/golioth/mtb-example-golioth)" application with the desired name "Psoc6Golioth" configured for the *CY8CPROTO-062-4343W* BSP into the specified working directory, *C:/mtb_projects*:

   ```
   project-creator-cli --board-id CY8CPROTO-062-4343W --app-id mtb-example-golioth --user-app-name Psoc6Golioth --target-dir "C:/mtb_projects"
   ```

**Note:** The project-creator-cli tool uses the `git clone` and `make getlibs` commands to fetch the repository and import the required libraries. For details, see the "Project creator tools" section of the [ModusToolbox&trade; software user guide](https://www.cypress.com/ModusToolboxUserGuide) (locally available at *{ModusToolbox&trade; software install directory}/docs_{version}/mtb_user_guide.pdf*).

To work with a different supported kit later, use the [Library Manager](https://www.infineon.com/ModusToolboxLibraryManager) to choose the BSP for the supported kit. You can invoke the Library Manager GUI tool from the terminal using the `make library-manager` command or use the Library Manager CLI tool "library-manager-cli" to change the BSP.

The "library-manager-cli" tool has the following arguments:

Argument | Description | Required/optional
---------|-------------|-----------
`--add-bsp-name` | Name of the BSP that should be added to the application | Required
`--set-active-bsp` | Name of the BSP that should be as active BSP for the application | Required
`--add-bsp-version`| Specify the version of the BSP that should be added to the application if you do not wish to use the latest from manifest | Optional
`--add-bsp-location`| Specify the location of the BSP (local/shared) if you prefer to add the BSP in a shared path | Optional

<br />

Following example adds the CY8CPROTO-062-4343W BSP to the already created application and makes it the active BSP for the app:

   ```
   library-manager-cli --project "C:/mtb_projects/Psoc6Golioth" --add-bsp-name CY8CPROTO-062-4343W --add-bsp-version "latest-v4.X" --add-bsp-location "local"

   library-manager-cli --project "C:/mtb_projects/Psoc6Golioth" --set-active-bsp APP_CY8CPROTO-062-4343W
   ```

</details>

<details><summary><b>In third-party IDEs</b></summary>

Use one of the following options:

- **Use the standalone [Project Creator](https://www.infineon.com/ModusToolboxProjectCreator) tool:**

   1. Launch Project Creator from the Windows Start menu or from *{ModusToolbox&trade; software install directory}/tools_{version}/project-creator/project-creator.exe*.

   2. In the initial **Choose Board Support Package** screen, select the BSP, and click **Next**.

   3. In the **Select Application** screen, select the appropriate IDE from the **Target IDE** drop-down menu.

   4. Click **Create** and follow the instructions printed in the bottom pane to import or open the exported project in the respective IDE.

<br />

- **Use command-line interface (CLI):**

   1. Follow the instructions from the **In command-line interface (CLI)** section to create the application.

   2. Export the application to a supported IDE using the `make <ide>` command.

   3. Follow the instructions displayed in the terminal to create or import the application as an IDE project.

For a list of supported IDEs and more details, see the "Exporting to IDEs" section of the [ModusToolbox&trade; software user guide](https://www.infineon.com/ModusToolboxUserGuide) (locally available at *{ModusToolbox&trade; software install directory}/docs_{version}/mtb_user_guide.pdf*).

</details>
