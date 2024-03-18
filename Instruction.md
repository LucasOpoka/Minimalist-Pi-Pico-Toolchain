## 1. Install git
sudo apt update
sudo apt install git

## 2. In home directory create pico folder and clone the Raspberry Pi Pico SDK repository there
cd ~
mkdir pico
cd pico
git clone -b master https://github.com/raspberrypi/pico-sdk.git

## 3. Set up PICO_SDK_PATH in your ~/.bashrc
echo "export PICO_SDK_PATH=/home/lucas/pico/pico-sdk" >> ~/.bashrc

## 4. Install SDK dependencies
sudo apt install -y cmake gcc-arm-none-eabi gcc g++

## 5. Install OPENOCD dependencies
sudo apt install -y gdb-multiarch automake autoconf build-essential texinfo libtool libftdi-dev libusb-1.0-0-dev pkg-config

## 6. Instal HIDAPI
sudo apt install libudev-dev
sudo apt-get install -y libhidapi-hidraw0
cd ~
git clone https://github.com/libusb/hidapi.git
cd hidapi
./bootstrap
./configure
make
sudo make install
cd ~
sudo rm -R hidapi

## 7. Install OPENOCD
cd ~
git clone https://github.com/raspberrypi/openocd.git --branch rp2040 --depth=1
git clone git://repo.or.cz/openocd.git
cd openocd
./bootstrap
./configure --enable-cmsis-dap
make
sudo make install

## 8. Create the file /etc/udev/rules.d/98-openocd.rules and add this content:
bash << EOF
sudo -i
echo "ACTION!=\"add|change\", GOTO=\"openocd_rules_end\"
SUBSYSTEM!=\"usb|tty|hidraw\", GOTO=\"openocd_rules_end\"
ATTRS{product}==\"*CMSIS-DAP*\", MODE=\"664\" GROUP=\"plugdev\"
LABEL=\"openocd_rules_end\"" >> /etc/udev/rules.d/thingy.rules
gpasswd -a lucas plugdev
udevadm control --reload
exit
EOF


## 9. Install VSCode
sudo snap install --classic code

## 10. Install required VSCode extensions
code --install-extension marus25.cortex-debug
code --install-extension ms-vscode.cmake-tools
code --install-extension ms-vscode.cpptools

## 11. Create folder for a test project "blinky" in our pico folder
cd ~
cd pico
mkdir blinky
cd blinky
code .

## In VSCode create a blinky.c file containing:
                #include "pico/stdlib.h"
                    int main()
                    {
                    gpio_init(25);
                    gpio_set_dir(25, GPIO_OUT);
                    while (true)
                    {
                        gpio_put(25, 1);
                        sleep_ms(500);
                        gpio_put(25, 0);
                        sleep_ms(500);
                    }
                }

## Copy pico_sdk_import.cmake to the root of the project
pico_sdk_import.cmake has to be copied into the root of
the project from pico/pico-sdk/external

## Then also in VSCode create a file called CMakeLists.txt containing:
cmake_minimum_required(VERSION 3.13)
include(pico_sdk_import.cmake)
project(blinky C CXX ASM)
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)
pico_sdk_init()
add_executable(blinky
blinky.c
)
target_link_libraries(blinky pico_stdlib)
pico_add_extra_outputs(blinky)





Resources:
https://www.hashdefineelectronics.com/compiling-and-installing-openocd-with-cmcsis-dap-support/
https://dev.to/admantium/raspberry-pico-simple-debugging-with-just-one-device-4ce7
https://raw.githubusercontent.com/raspberrypi/pico-setup/master/pico_setup.sh
