# 🔨 ZMK Build Instructions - Custom 60% Keyboard

## Prerequisites

### 1. Install Required Tools

```bash
# Install Python 3 and pip
sudo apt update
sudo apt install python3 python3-pip

# Install west (Zephyr build tool)
pip3 install --user west

# Add ~/.local/bin to PATH if not already added
echo 'export PATH=$PATH:~/.local/bin' >> ~/.bashrc
source ~/.bashrc

# Install additional dependencies
sudo apt install --no-install-recommends git cmake ninja-build gperf \
  ccache dfu-util device-tree-compiler wget \
  python3-dev python3-pip python3-setuptools python3-tk python3-wheel xz-utils file \
  make gcc gcc-multilib g++-multilib libsdl2-dev libmagic1
```

### 2. Install Zephyr SDK

```bash
# Download and install Zephyr SDK (required for ARM compilation)
cd ~
wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.8/zephyr-sdk-0.16.8_linux-x86_64.tar.xz
tar xf zephyr-sdk-0.16.8_linux-x86_64.tar.xz
cd zephyr-sdk-0.16.8
./setup.sh
```

## Building ZMK Firmware

### Method 1: Local Build (Recommended)

Navigate to your ZMK config directory:

```bash
cd /home/hashim/Projects/60_Percent_Keyboard/Firmware/zmk-config
```

#### Step 1: Initialize ZMK Workspace

```bash
# Initialize west workspace
west init -l config/

# Update and install dependencies
west update

# Install Python dependencies
west zephyr-export
pip3 install --user -r zephyr/scripts/requirements.txt
```

#### Step 2: Build the Firmware

```bash
# Build firmware for your keyboard
west build -p -b seeeduino_xiao_ble -- -DSHIELD=id1

# Alternative: Build with specific config location
west build -p -b seeeduino_xiao_ble boards/shields/id1
```

#### Step 3: Locate Built Firmware

```bash
# The firmware will be located at:
ls build/zephyr/zmk.uf2

# Copy to a convenient location
cp build/zephyr/zmk.uf2 ~/custom60_firmware.uf2
```

### Method 2: GitHub Actions Build

If local build fails, you can use GitHub Actions:

#### Step 1: Push to GitHub

```bash
# Add all files to git
git add .
git commit -m "Add custom 60% keyboard configuration"
git push origin main
```

#### Step 2: Download from Actions

1. Go to your GitHub repository
2. Click on "Actions" tab
3. Wait for build to complete
4. Download the firmware artifact
5. Extract the `zmk.uf2` file

## Flashing the Firmware

### Step 1: Enter Bootloader Mode

1. **Double-tap** the reset button on your XIAO nRF52840
2. The board should appear as a USB drive (usually named "XIAO-SENSE")

### Step 2: Flash Firmware

```bash
# Simply copy the UF2 file to the mounted drive
cp ~/custom60_firmware.uf2 /media/$(whoami)/XIAO-SENSE/

# Or drag and drop the file in your file manager
```

### Step 3: Verify Flash

- The board will automatically reboot
- LED should indicate successful flash
- Test basic functionality

## Troubleshooting Build Issues

### Common Build Errors and Solutions

#### Error: "west command not found"

```bash
# Ensure west is in PATH
export PATH=$PATH:~/.local/bin
# Or reinstall west
pip3 install --user --force-reinstall west
```

#### Error: "No such file or directory: arm-zephyr-eabi-gcc"

```bash
# Install/reinstall Zephyr SDK
cd ~/zephyr-sdk-0.16.8
./setup.sh -c
```

#### Error: "shield 'id1' not found"

```bash
# Verify shield files exist
ls boards/shields/id1/
# Should show: id1.overlay, id1.keymap, id1.conf, etc.
```

#### Error: PCF8575 or I2C related errors

⚠️ **Expected Issue**: ZMK doesn't have built-in PCF8575 support. You may see:

- I2C configuration warnings
- Missing kscan driver errors
- GPIO expander issues

**Solutions**:

1. **Implement custom PCF8575 driver** (advanced)
2. **Switch to direct GPIO connections**
3. **Use QMK firmware instead** (has PCF8575 support)

### Build Clean/Reset

If you encounter persistent issues:

```bash
# Clean build directory
west build -t clean

# Or delete and rebuild
rm -rf build/
west build -p -b seeeduino_xiao_ble -- -DSHIELD=id1
```

## Testing Your Firmware

### Basic Functionality Tests

1. **USB Connection**: Keyboard should be detected by OS
2. **Key Presses**: Test basic keys (may not work without custom driver)
3. **Bluetooth**: Should appear in Bluetooth device list
4. **RGB LEDs**: Test if configured LEDs light up
5. **OLED Display**: Check if display shows information

### Expected Limitations

⚠️ **Without custom PCF8575 driver**:

- Matrix scanning won't work properly
- Keys may not register
- I2C expanders won't function

## Next Steps After Build

1. **Test basic connectivity** (USB/Bluetooth)
2. **Verify GPIO functionality** where possible
3. **Develop custom PCF8575 driver** or
4. **Consider switching to QMK** for proven I2C expander support

## Build Success Indicators

✅ **Successful build shows**:

```
[100%] Built target zephyr
-- west build: build completed successfully
```

✅ **Generated files**:

- `build/zephyr/zmk.uf2` (main firmware)
- `build/zephyr/zephyr.elf` (debug info)

🎉 **You're ready to flash!**

---

Need help? Check the [ZMK Discord](https://discord.gg/8cfMkQksSB) for community support.
