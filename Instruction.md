## 1. Install git
sudo apt update
sudo apt install git

## 2. Clone the Minimal Pi Pico Toolchain repository to home as pico, then clone the Raspberry Pi Pico SDK repository there
cd ~
git clone https://github.com/LucasOpoka/Minimalist-Pi-Pico-Toolchain.git pico

cd pico
git clone -b master https://github.com/raspberrypi/pico-sdk.git

## 3. Install SDK dependencies
sudo apt install -y cmake gcc-arm-none-eabi gcc g++

## 4. Install OPENOCD dependencies
sudo apt install -y gdb-multiarch automake autoconf build-essential texinfo libtool libftdi-dev libusb-1.0-0-dev pkg-config

## 5. Instal HIDAPI
sudo apt install libudev-dev libhidapi-hidraw0
cd ~
git clone https://github.com/libusb/hidapi.git
cd hidapi
./bootstrap
./configure
make
sudo make install
cd ~
sudo rm -R hidapi

## 6. Install OPENOCD
cd ~
git clone https://github.com/raspberrypi/openocd.git --branch rp2040 --depth=1
//git clone git://repo.or.cz/openocd.git
cd openocd
./bootstrap
./configure --enable-cmsis-dap
make
sudo make install


## 7. Define PICO_SDK_PATH and OPENOCD_TCL_PATH
echo "export PICO_SDK_PATH=/home/$USER/pico/pico-sdk" >> ~/.bashrc
export PICO_SDK_PATH=/home/$USER/pico/pico-sdk

echo "export OPENOCD_TCL_PATH=/home/$USER/openocd/tcl" >> ~/.bashrc
export OPENOCD_TCL_PATH=/home/$USER/openocd/tcl

source ~/.bashrc


## 8. Create the file /etc/udev/rules.d/98-openocd.rules and add this content:
bash << EOF
sudo -i
echo "ACTION!=\"add|change\", GOTO=\"openocd_rules_end\"
SUBSYSTEM!=\"usb|tty|hidraw\", GOTO=\"openocd_rules_end\"
ATTRS{product}==\"*CMSIS-DAP*\", MODE=\"664\" GROUP=\"plugdev\"
LABEL=\"openocd_rules_end\"" >> /etc/udev/rules.d/thingy.rules
gpasswd -a $USER plugdev
udevadm control --reload
exit
EOF


## 9. Install VSCode
sudo snap install --classic code


## 10. Install required VSCode extensions
code --install-extension marus25.cortex-debug
code --install-extension ms-vscode.cmake-tools
code --install-extension ms-vscode.cpptools



Resources:
https://www.hashdefineelectronics.com/compiling-and-installing-openocd-with-cmcsis-dap-support/
https://dev.to/admantium/raspberry-pico-simple-debugging-with-just-one-device-4ce7
https://raw.githubusercontent.com/raspberrypi/pico-setup/master/pico_setup.sh
