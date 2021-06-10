---
title: Connect an NXP MIMXRT1060-EVK to Azure IoT Central quickstart
description: Use Azure RTOS embedded software to connect an NXP MIMXRT1060-EVK device to Azure IoT and send telemetry.
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.devlang: c
ms.topic: quickstart
ms.date: 06/04/2021
---

# Quickstart: Connect an NXP MIMXRT1060-EVK Evaluation kit to IoT Central

**Applies to**: [Embedded device development](about-iot-develop.md#embedded-device-development)<br>
**Total completion time**:  30 minutes

[![Browse code](media/common/browse-code.svg)](https://github.com/azure-rtos/getting-started/tree/master/NXP/MIMXRT1060-EVK)

In this quickstart, you use Azure RTOS to connect the NXP MIMXRT1060-EVK Evaluation kit (hereafter, the NXP EVK) to Azure IoT.

You will complete the following tasks:

* Install a set of embedded development tools for programming an NXP EVK in C
* Build an image and flash it onto the NXP EVK
* Use Azure IoT Central to create cloud components, view properties, view device telemetry, and call direct commands

## Prerequisites

* A PC running Linux or Microsoft Windows 10
* [Git](https://git-scm.com/downloads) for cloning the repository
* Hardware

    > * The [NXP MIMXRT1060-EVK](https://www.nxp.com/design/development-boards/i-mx-evaluation-and-development-boards/mimxrt1060-evk-i-mx-rt1060-evaluation-kit:MIMXRT1060-EVK) (NXP EVK)
    > * USB 2.0 A male to Micro USB male cable
    > * Wired Ethernet access
    > * Ethernet cable

## Prepare the development environment

To set up your development environment, first you clone a GitHub repo that contains all the assets you need for the quickstart. Then you install a set of programming tools.

### Clone the repo for the quickstart

Clone the following repo to download all sample device code, setup scripts, and offline versions of the documentation. If you previously cloned this repo in another quickstart, you don't need to do it again.

To clone the repo, run the following command:

```shell
git clone --recursive https://github.com/azure-rtos/getting-started.git
```

### Install the tools on Windows

> [!NOTE]
> Following the instructions in this section will install the following tools:
> * [Cella](#): Package Manager
> * [CMake](https://cmake.org): Build
> * [ARM GCC](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm): Compile
> * [Termite](https://www.compuphase.com/software_termite.htm): Monitor serial port output for connected devices

#### Using PowerShell

1. Open an Administrator PowerShell terminal and enable execution of PowerShell scripts:

    ```shell
    Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope LocalMachine
    ```

1. Open a new PowerShell terminal.

1. Navigate to the following path in the repo:

    > *getting-started\NXP\MIMXRT1060-EVK*

1. Install Cella

    ```shell
    iex (iwr -useb aka.ms/cella.ps1)
    ```

1. Download, install, and activate the tools:

    ```shell
    cella update; cella activate
    ```

1. Run the following code to confirm that CMake version 3.20 or later is installed:

    ```shell
    cmake --version
    ```

1. Use this terminal to complete the remaining programming tasks in the quickstart.

> [!NOTE]
> To activate these tools in any new PowerShell terminals you create, run `~/.cella/cella.ps1 activate` from the project folder.

#### Using Windows CMD

1. Open a new Windows CMD terminal.

1. Navigate to the following path in the repo:

    > *getting-started\NXP\MIMXRT1060-EVK*

1. Install Cella

    ```shell
    curl -L aka.ms/cella.cmd -o cella.cmd && cella.cmd
    ```

1. Download, install, and activate the tools:

    ```shell
    cella update && cella activate
    ```

1. Run the following code to confirm that CMake version 3.20 or later is installed:

    ```shell
    cmake --version
    ```

1. Use this terminal to complete the remaining programming tasks in the quickstart.

> [!NOTE]
> To activate these tools in any new CMD terminals you create, run `%userprofile%\.cella\cella.cmd` from the project folder.

#### Install the tools on Linux

> [!NOTE]
> Following the instructions in this section will install the following tools:
> * [Cella](#): Package Manager
> * [CMake](https://cmake.org): Build
> * [ARM GCC](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm): Compile
> * [screen](https://www.gnu.org/software/screen/): Monitor serial port output for connected devices
> * [curl](https://curl.se/): Network download utility
> * [libncurses5](https://invisible-island.net/ncurses/): Textual UI library used by ARM GCC

To install the tools:

1. Open a new `bash` terminal.

1. Navigate to the following path in the repo:

    > *getting-started\NXP\MIMXRT1060-EVK*

1. Install `screen`, `curl`, and `libncurses` using your package manager.
    > For example, on Ubuntu, run `sudo apt install screen curl libncurses5`. On Fedora, run `sudo yum install screen curl libncurses5`.
  
1. Install Cella

    ```shell
    . <(curl aka.ms/cella.sh -L )
    ```

1. Download, install, and activate the tools:

    ```shell
    cella update && cella activate
    ```

1. Run the following code to confirm that CMake version 3.20 or later is installed:

    ```shell
    cmake --version
    ```

1. Use this terminal to complete the remaining programming tasks in the quickstart.

> [!NOTE]
> To activate these tools in any new terminals you create, run `. ~/.cella/cella activate` from the project folder.

[!INCLUDE [iot-develop-embedded-create-central-app-with-device](../../includes/iot-develop-embedded-create-central-app-with-device.md)]

## Prepare the device

To connect the NXP EVK to Azure, you'll modify a configuration file for Azure IoT settings, rebuild the image, and flash the image to the device.

### Add configuration

1. Open the following file in a text editor:

    > *getting-started\NXP\MIMXRT1060-EVK\app\azure_config.h*

1. Set the Azure IoT device information constants to the values that you saved after you created Azure resources.

    |Constant name|Value|
    |-------------|-----|
    |`IOT_DPS_ID_SCOPE` |{*Your ID scope value*}|
    |`IOT_DPS_REGISTRATION_ID` |{*Your Device ID value*}|
    |`IOT_DEVICE_SAS_KEY` |{*Your Primary key value*}|

1. Save and close the file.

### Build the image

In your terminal, run the following commands to build the image:

```shell
cmake -Bbuild -GNinja -DCMAKE_TOOLCHAIN_FILE="../../cmake/arm-gcc-cortex-m7.cmake"
cmake --build build
```

After the build completes, confirm that the binary file was created in the following path:

> *getting-started\NXP\MIMXRT1060-EVK\build\app\mimxrt1060_azure_iot.bin*

### Flash the image

1. On the NXP EVK, locate the **Reset** button, the Micro USB port, and the Ethernet port.  You use these components in the following steps. All three are highlighted in the following picture:

    :::image type="content" source="media/quickstart-devkit-nxp-mimxrt1060-evk/nxp-evk-board.png" alt-text="Locate key components on the NXP EVK board":::

1. Connect the Micro USB cable to the Micro USB port on the NXP EVK, and then connect it to your computer. After the device powers up, a solid green LED shows the power status.
1. Use the Ethernet cable to connect the NXP EVK to an Ethernet port.
1. In your file manager, find the binary file that you created in the previous section.
1. Copy the binary file *mimxrt1060_azure_iot.bin*
1. In your file manager, find the NXP EVK device connected to your computer. The device appears as a drive on your system with the drive label **RT1060-EVK**.
1. Paste the binary file into the root folder of the NXP EVK. Flashing starts automatically and completes in a few seconds.

    > [!NOTE]
    > During the flashing process, a red LED blinks rapidly on the NXP EVK.

### Confirm device connection details using Termite

On Windows, you can use the **Termite** app to monitor communication and confirm that your device is set up correctly.

1. Start **Termite**.

    ```shell
    termite
    ```

    > [!TIP]
    > If you have issues getting your device to initialize or connect after flashing, see [Troubleshooting](https://github.com/azure-rtos/getting-started/blob/master/docs/troubleshooting.md).
1. Select **Settings**.
1. In the **Serial port settings** dialog, check the following settings and update if needed:
    * **Baud rate**: 115,200
    * **Port**: The port that your NXP EVK is connected to. If there are multiple port options in the dropdown, you can find the correct port to use. Open Windows **Device Manager**, and view **Ports** to identify which port to use.

    :::image type="content" source="media/quickstart-devkit-nxp-mimxrt1060-evk/termite-settings.png" alt-text="Confirm settings in the Termite app":::

1. Select OK.
1. Press the **Reset** button on the device. The button is labeled on the device and located near the Micro USB connector.
1. In the **Termite** app, check the following checkpoint values to confirm that the device is initialized and connected to Azure IoT.

    ```output
    Starting Azure thread

    Initializing DHCP
	    IP address: 192.168.0.19
	    Mask: 255.255.255.0
	    Gateway: 192.168.0.1
    SUCCESS: DHCP initialized

    Initializing DNS client
	    DNS address: 75.75.75.75
    SUCCESS: DNS client initialized

    Initializing SNTP client
	    SNTP server 0.pool.ntp.org
	    SNTP IP address: 108.62.122.57
	    SNTP time update: May 20, 2021 19:41:20.319 UTC 
    SUCCESS: SNTP initialized

    Initializing Azure IoT DPS client
	    DPS endpoint: global.azure-devices-provisioning.net
	    DPS ID scope: ***
	    Registration ID: mydevice
    SUCCESS: Azure IoT DPS client initialized

    Initializing Azure IoT Hub client
	    Hub hostname: ***.azure-devices.net
	    Device id: mydevice
	    Model id: dtmi:azurertos:devkit:gsg;1
    Connected to IoT Hub
    SUCCESS: Azure IoT Hub client initialized
    ```

Keep Termite open to monitor device output in the following steps.

### Confirm device connection details using screen

On Linux, you can use the `screen` utility to monitor communication and confirm that your device is set up correctly.

1. Start `screen`. The path to the device may differ on your system. Run the following command to list the available devices by id: `ls -l /dev/serial/by-id`

    ```shell
    sudo screen /dev/ttyACM0 115200
    ```

    > [!TIP]
    > If you have issues getting your device to initialize or connect after flashing, see [Troubleshooting](https://github.com/azure-rtos/getting-started/blob/master/docs/troubleshooting.md).

1. Press the **Reset** button on the device. The button is labeled on the device and located near the Micro USB connector.

1. Check that `screen` outputs the following checkpoint values to confirm that the device is initialized and connected to Azure IoT.

    ```output
    Starting Azure thread

    Initializing DHCP
	    IP address: 192.168.0.19
	    Mask: 255.255.255.0
	    Gateway: 192.168.0.1
    SUCCESS: DHCP initialized

    Initializing DNS client
	    DNS address: 75.75.75.75
    SUCCESS: DNS client initialized

    Initializing SNTP client
	    SNTP server 0.pool.ntp.org
	    SNTP IP address: 108.62.122.57
	    SNTP time update: May 20, 2021 19:41:20.319 UTC 
    SUCCESS: SNTP initialized

    Initializing Azure IoT DPS client
	    DPS endpoint: global.azure-devices-provisioning.net
	    DPS ID scope: ***
	    Registration ID: mydevice
    SUCCESS: Azure IoT DPS client initialized

    Initializing Azure IoT Hub client
	    Hub hostname: ***.azure-devices.net
	    Device id: mydevice
	    Model id: dtmi:azurertos:devkit:gsg;1
    Connected to IoT Hub
    SUCCESS: Azure IoT Hub client initialized
    ```

Keep `screen` running to monitor device output in the following steps.

## Verify the device status

To view the device status in IoT Central portal:
1. From the application dashboard, select **Devices** on the side navigation menu.
1. Confirm that the **Device status** is updated to **Provisioned**.
1. Confirm that the **Device template** is updated to **Getting Started Guide**.

    :::image type="content" source="media/quickstart-devkit-nxp-mimxrt1060-evk/iot-central-device-view-status.png" alt-text="View device status in IoT Central":::

## View telemetry

With IoT Central, you can view the flow of telemetry from your device to the cloud.

To view telemetry in IoT Central portal:

1. From the application dashboard, select **Devices** on the side navigation menu.
1. Select the device from the device list.
1. View the telemetry as the device sends messages to the cloud in the **Overview** tab.
1. The temperature is measured from the MCU wafer.

    :::image type="content" source="media/quickstart-devkit-nxp-mimxrt1060-evk/iot-central-device-telemetry.png" alt-text="View device telemetry in IoT Central":::

    > [!NOTE]
    > You can also monitor telemetry from the device by using the Termite app.

## Call a direct method on the device

You can also use IoT Central to call a direct method that you've implemented on your device. Direct methods have a name, and can optionally have a JSON payload, configurable connection, and method timeout. In this section, you call a method that enables you to turn an LED on or off.

To call a method in IoT Central portal:

1. Select the **Command** tab from the device page.
1. In the **State** dropdown, select **True**, and then select **Run**. There will be no change on the device as there isn't an available LED to toggle; however, you can view the output in Termite to monitor the status of the methods.

    :::image type="content" source="media/quickstart-devkit-nxp-mimxrt1060-evk/iot-central-invoke-method.png" alt-text="Call a direct method on a device":::

1. In the **State** dropdown, select **False**, and then select **Run**.

## View device information

You can view the device information from IoT Central.

Select **About** tab from the device page.

:::image type="content" source="media/quickstart-devkit-nxp-mimxrt1060-evk/iot-central-device-about.png" alt-text="View information about the device in IoT Central":::

## Troubleshoot and debug

If you experience issues building the device code, flashing the device, or connecting, see [Troubleshooting](https://github.com/azure-rtos/getting-started/blob/master/docs/troubleshooting.md).

For debugging the application, see [Using Visual Studio Code with the NXP MIMXRT1060-EVK Evaluation Kit](https://github.com/azure-rtos/getting-started/blob/master/NXP/MIMXRT1060-EVK/vscode.md).

## Clean up resources

If you no longer need the Azure resources created in this quickstart, you can delete them from the IoT Central portal.

To remove the entire Azure IoT Central sample application and all its devices and resources:
1. Select **Administration** > **Your application**.
1. Select **Delete**.

## Next steps

In this quickstart, you built a custom image that contains Azure RTOS sample code, and then flashed the image to the NXP EVK device. You also used the IoT Central portal to create Azure resources, connect the NXP EVK securely to Azure, view telemetry, and send messages.

As a next step, explore the following articles to learn more about using the IoT device SDKs to connect devices to Azure IoT. 

> [!div class="nextstepaction"]
> [Connect a simulated device to IoT Central](quickstart-send-telemetry-central.md)
> [!div class="nextstepaction"]
> [Connect a simulated device to IoT Hub](quickstart-send-telemetry-iot-hub.md)

> [!IMPORTANT]
> Azure RTOS provides OEMs with components to secure communication and to create code and data isolation using underlying MCU/MPU hardware protection mechanisms. However, each OEM is ultimately responsible for ensuring that their device meets evolving security requirements.

