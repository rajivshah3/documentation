# Get started with the PlatformIO integration

**PlatformIO is an ecosystem and development platform for embedded devices. It is flexible and has a lot of helpful integrated tools for the most common IDEs. This makes it a good platform to get into more advanced embedded development**

## Hardware

To complete this guide you need an ESP32 development board. 
You can also use a STM32F7 development board, but the guide will differ a bit for you.
Read the [introduction into PlatformIO](#introduction-into-platformIO) carefully and 
You can use Windows, Mac or Linux as operating system for this guide.

## Introduction into platformIO

In order to be able to follow this guide, you need to understand platformIO a bit.
PlatformIO is a development ecosystem for several different frameworks, IDEs and boards.
Therefore it is highly flexible to adapt to different environments.

When you start your project, it needs to get initialized for your specific board and IDE.
If you don't use Visual Studio Code, make sure to [follow the documentation](https://docs.platformio.org/en/latest/integration/ide/index.html) 
for your IDE before you start following the guide.
PlatformIO supports different frameworks, such as the Arduino framework (part of the Arduino IDE), [mbed OS](https://www.mbed.com/en/platform/mbed-os/) or [Zephyr OS](https://www.zephyrproject.org/).
You are free to choose the framework you are going to use. Arduino is the most simple one, while RTOS based frameworks (such as mbed or Zephyr)
give you more flexibility and advanced functionality. We are going to focus our guides on the Arduino framework, if possible.
Not all boards and microcontroller are supported by all frameworks. To check, if your board supports Arduino, you
need to [search for your specific board](https://platformio.org/boards). 
The column framework lists all available frameworks for your board.
If you are going to use another framework than Arduino, please check the [documentation for your framework](https://docs.platformio.org/en/latest/frameworks/index.html#frameworks) before.
The structure of the code will differ from the structure used in our guides.
We have some [examples](#example-applications) available for different frameworks.

## Step 1. Install platformIO

[Install platformIO](https://platformio.org/install) for your operating system and your IDE.
If you don't have an IDE yet, we recommend to use [Visual Studio Code.](https://code.visualstudio.com/)
It is a free and [open source IDE](https://github.com/Microsoft/vscode), written in Typescript.

## Step 2. Install the platformIO plugin

If you use the Visual Studio Code, you need to install the [platformIO plugin for it.](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide)

## Step 3. Create your project

Open Visual Studio Code and [create your project](https://docs.platformio.org/en/latest/integration/ide/vscode.html#setting-up-the-project).
Give your project a name, select `Arduino` as framework and select the board you are going to use.
If you are not sure which board you need to select, 
there is a [web based board search](https://platformio.org/boards) with more filters.

## Step 4. Add iota as dependency to your platformio.ini file

Edit the `platformio.ini` file and add the following to the start of it:
```
[external_libs]
lib_deps_external =
    https://github.com/oopsmonk/iota_common.git#pio_lib
    https://github.com/troydhanson/uthash.git#1124f0a70b0714886402c3c0df03d037e3c4d57a
    https://github.com/oopsmonk/XKCP.git#pio_keccakp1600
```

After the board definition, you have to add the following:

```
build_flags =
    -I${PROJECT_LIBDEPS_DIR}/${PIOENV}/Keccak/lib/common
    -I${PROJECT_LIBDEPS_DIR}/${PIOENV}/Keccak/lib/high/Keccak
    -I${PROJECT_LIBDEPS_DIR}/${PIOENV}/Keccak/lib/high/Keccak/FIPS202
    -I${PROJECT_LIBDEPS_DIR}/${PIOENV}/Keccak/lib/low/KeccakP-1600/Reference

; Library options
lib_deps =
    ${external_libs.lib_deps_external}
```

Your `platformio.ini` file has to look similar to this one. The board might differ.

```
[external_libs]
lib_deps_external =
    https://github.com/oopsmonk/iota_common.git#pio_lib
    https://github.com/troydhanson/uthash.git#1124f0a70b0714886402c3c0df03d037e3c4d57a
    https://github.com/oopsmonk/XKCP.git#pio_keccakp1600

[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino

build_flags =
    -I${PROJECT_LIBDEPS_DIR}/${PIOENV}/Keccak/lib/common
    -I${PROJECT_LIBDEPS_DIR}/${PIOENV}/Keccak/lib/high/Keccak
    -I${PROJECT_LIBDEPS_DIR}/${PIOENV}/Keccak/lib/high/Keccak/FIPS202
    -I${PROJECT_LIBDEPS_DIR}/${PIOENV}/Keccak/lib/low/KeccakP-1600/Reference

; Library options
lib_deps =
    ${external_libs.lib_deps_external}
```

## Step 5. Start developing

You can find the source code in the `src` directory.
It supports the commonly known structure of the Arduino IDE. There are two functions `setup()` and `loop()`.
If you already have an application written in the Arduino IDE, you are able to port it to PlatformIO in a few steps.

## Step 6. Deploy your application

Once you are done with developing your application, you are able to [compile and upload](https://docs.platformio.org/en/latest/integration/ide/vscode.html#setting-up-the-project) it to your board.
You can find two small buttons on the left bottom corner. 
![Deploy and Upload](../images/vscode_deploy.png)

## Example applications

We have some example applications available for different frameworks. 
[mbed OS](https://github.com/iota-community/iota_c_platformIO/blob/mbed_stm32f746zg/src/my_app.cpp),
[arduino](https://github.com/iota-community/iota_c_platformIO/blob/arduino_esp32/src/main.cpp) or
[esp-idf (freeRTOS)](https://github.com/iota-community/iota_c_platformIO/tree/esp_idf_esp32/src)
If you need more information about the libraries API,
 check out the [iota.c API reference.](https://github.com/iotaledger/iota.c#api-reference)


