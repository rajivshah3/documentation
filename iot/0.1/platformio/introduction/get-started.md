# Get started with the PlatformIO integration

**PlatformIO is an ecosystem and development platform for embedded devices. It is flexible and has a lot of helpful integrated tools for the most common IDEs. This makes it a good platform to get into more advanced embedded development**

## Hardware

To complete this guide you need an ESP32 or STM32F7. 
You can use Windows, Mac or Linux as operating system for this guide.

## Step 1. Install platformIO

[Install platformIO](https://platformio.org/install) for your operating system and your IDE.
If you don't have an IDE yet, we recommend to use [visual studio code.](https://code.visualstudio.com/)
It is a free and [open source IDE](https://github.com/Microsoft/vscode), written in Typescript.

## Step 2. Create your project

Create a directory and [initialize the project](https://docs.platformio.org/en/latest/core/userguide/project/cmd_init.html).
You can search for your board id on the [platformIO board search page.](https://platformio.org/boards)
Just click on the name of your board to get more information and the board ID.
Also don't forget to use the `` --ide`` option to initialize the project for your specific IDE.
The command you use should look similar to this one: 
```bash
pio project init --ide clion --board esp32dev
```

## Step 3. Add iota as dependency to your platformio.ini file

Add the following to the start of your platformio.ini file.
```
[external_libs]
lib_deps_external =
    https://github.com/oopsmonk/iota_common.git#pio_lib
    https://github.com/troydhanson/uthash.git#1124f0a70b0714886402c3c0df03d037e3c4d57a
    https://github.com/oopsmonk/XKCP.git#pio_keccakp1600
```

and the following after the board definition.

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

Your `platformio.ini` file has to look similar to this

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

## Step 4. Update your development environment

After adding the dependency to your `platformio.ini`, you need to update your development environment.
Just execute the command you used in step 2 again. You might need to restart or reload your IDE. 
For more details about the specific process of your IDE, 
take a look into the [platformIO documentation for your IDE.](https://docs.platformio.org/en/latest/integration/ide/index.html)

## Step 5. Start developing

If your board supports the Arduino environment, you are able to simply add a `main.cpp` file under the src directory.
It supports the commonly known structure of the Arduino IDE. There are two functions `setup()` and `loop()`.
If you already have an application written in the Arduino IDE, you are able to port it to PlatformIO in a few steps.
Before you do so, you have to make sure that PlatformIO uses the Arduino framework environment in your project.
Check your `platformio.ini` file. There is a framework option under the board definition.
If it already is equal arduino `framework = arduino`, you are ready to go. If not, you have to change it to arduino. 
Before you do so, you have to check the [board search page](https://platformio.org/boards), 
if your board supports the arduino framework.

PlatformIO has several other options, if the arduino framework is not available for your board.
You have to [check the support of your board](https://platformio.org/boards) before. PlatformIO has [documentation for all available frameworks.](https://docs.platformio.org/en/latest/frameworks/index.html#frameworks)

## Step 6. Deploy your application

Deploying your application in your IDE is quite simple. With the initialization of PlatformIO, 
it also adds some targets to the build system in your IDE. [Check out the documentation for your IDE.](https://docs.platformio.org/en/latest/integration/ide/index.html)
In most cases, you are able to just click the build and then the upload/program button.
If you want to check the serial monitor of your board, you have to add the baud speed to the monitor target.
Just add `--baud=YOUR_SPEED` as option. The baud speed is specified in your application or framework.
Check out the [documentation page for your used framework.](https://docs.platformio.org/en/latest/frameworks/index.html#frameworks)


## Example applications

We have some example applications available for different frameworks. 
[mbed OS](https://github.com/iota-community/iota_c_platformIO/blob/mbed_stm32f746zg/src/my_app.cpp),
[arduino](https://github.com/iota-community/iota_c_platformIO/blob/arduino_esp32/src/main.cpp) or
[esp-idf (freeRTOS)](https://github.com/iota-community/iota_c_platformIO/tree/esp_idf_esp32/src)
If you need more information about the libraries API,
 check out the [iota.c API reference.](https://github.com/iotaledger/iota.c#api-reference)


