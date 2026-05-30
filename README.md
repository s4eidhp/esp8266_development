# ESP8266 Development
In this gist i want to sotre all related contents and documents which can help some one to start with ESP8266 development and deep dive into bits and registers. I also want to introduce different SDKs which can be used for programming and also bare-metal approach too.

## Specifications
- Processor: L106 32-bit RISC microprocessor core based on the Tensilica Diamond Standard 106Micro running at 80 or 160 MHz
- Memory of 160 Kbytes of RAM which are segmented into:
    - 32 KB instruction RAM (iRAM)
    - 32 KB instruction cache RAM
    - 96 KB of dRAM which are segmented into 80 KB dRAM for SDK and heap memory, and 16 KB for ROM
- External QSPI flash: up to 16 MiB is supported (512 KiB to 4 MiB typically included)
- IEEE 802.11 b/g/n Wi-Fi
    - Integrated TR switch, balun, LNA, power amplifier and matching network
    - WEP or WPA/WPA2 authentication, or open networks
- 17 GPIO pins
- Serial Peripheral Interface Bus (SPI)
- I²C (software implementation)
- I²S interfaces with DMA (sharing pins with GPIO)
- UART on dedicated pins, plus a transmit-only UART can be enabled on GPIO2
- 10-bit ADC (successive approximation ADC)

## SDKs
There is plenty of SDKs but the most important ones are:
- [Official FreeRTOS based](https://github.com/espressif/esp8266_rtos_sdk)
- [Official NonOS](https://github.com/espressif/ESP8266_NONOS_SDK)
- [ESP-Open-SDK based on GCC](https://github.com/pfalcon/esp-open-sdk)
- [Unofficial Development Kit](https://github.com/CHERTS/esp8266-devkit)
- [ESP-Open-RTOS](https://github.com/SuperHouse/esp-open-rtos)

In this guide we deep dive into nonOS and RTOS SDKs.

### NonOS SDK

1. Prepare Development Environment
```bash
git clone https://github.com/espressif/ESP8266_NONOS_SDK.git

# for newer version of ubuntu (from 2018 up to now) we need to download python2 source code and build it manually
mkdir ~/python2_src ~/.localpython2
cd ~/python2_src
wget http://www.python.org/ftp/python/2.7.18/Python-2.7.18.tgz      # latest python2 stable release
tar -zxvf Python-2.7.18.tgz
cd python-2.7.18
./configure --prefix=/home/${USER}/.localpython2
make
make install

# Change Makefile in main directory of SDK
PYTHON=${HOME}/.localpython2/bin/python2.7
```

2. Create Project & Build
```bash
# copy one example from SDK
cp -r examples/peripheral_test ~/uart_test
cd ~/uart_test

# this script build program and take 5 steps to configure
# 1. Config Bootloader version (none)
# 2. Which binaries should be generate (eagle.flash.bin+eagle.irom0text.bin)
# 3. Set SPI flash speed (40MHz)
# 4. Set SPI flash mode (DIO)
# 5. Set SPI flash size & map (4096KB(512KB + 512KB))
# Note: consider 2 OTA segment each of them is 512KB
./gen_misc
```

3. Flash

    There is two way for download firmware:
    1. Flash download tool
        - Download it from here: https://dl.espressif.com/public/flash_download_tool.zip
        - Always erase chip at first and then program it 
    2. ESPTool
    ```bash
    esptool -p /dev/ttyUSB0 
    esptool -p /dev/ttyUSB0 \
            -b 115200 \
            --after hard_reset \
            write_flash \
            --flash_mode dio \
            --flash_size 4MB \
            --flash_freq 40m \
            0x0 eagle.flash.bin \
            0x10000 eagle.irom0text.bin \
            0x3fb000 blank.bin \
            0x3fc000 esp_init_data_default_v08.bin \
            0x3fe000 blank.bin
    ```

## ToDo
- FOTA vs Non-FOTA development
- Single file image and patch creation for a device in production
- Getting start with different peripherals
- Compare Flash size between NonOS and RTOS based SDKs
- VGA display connection
- Overclocking CPU
- Linux port for ESP8266