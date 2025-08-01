# ZMK Firmware - Custom 60% Keyboard

**⚠️ ADVANCED CONFIGURATION REQUIRED** - This ZMK configuration requires significant custom driver development.

## ⚠️ Important Limitation

**ZMK does not have built-in PCF8575 kscan (key scanning) driver support.** You will need to:

1. **Implement a custom PCF8575 kscan driver**, OR
2. **Use direct GPIO connections instead of I2C expanders**, OR
3. **Switch to QMK firmware** (which has proven I2C expander support)

## Why ZMK is Challenging for This Build

ZMK's strength is **Bluetooth and power efficiency**, but it has limited I2C expander ecosystem compared to QMK. The PCF8575 chips require custom driver development.

## Required Manual Revisions

### 🔧 **CRITICAL Hardware Assignments**

#### 1. I2C Pin Configuration (id1.overlay)

```dts
&xiao_i2c {
    // UPDATE: Verify these are correct XIAO nRF52840 I2C pins
    // Default: P0.04 (SDA), P0.05 (SCL)
}
```

#### 2. PCF8575 I2C Addresses (id1.overlay)

```dts
pcf8575_main: pcf8575@20 {
    reg = <0x20>; // ❗ VERIFY: Your main keyboard expander address
};

pcf8575_numpad: pcf8575@21 {
    reg = <0x21>; // ❗ VERIFY: Your numpad expander address
};
```

#### 3. PCF8575 Interrupt Pins (id1.overlay)

```dts
// ❗ UPDATE: Connect these to PCF8575 /INT pins
row-gpios = <&gpio0 2 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>; // Main INT
row-gpios = <&gpio0 3 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>; // Numpad INT
```

### 🎛️ **Encoder Configuration** (id1.overlay)

#### 4. Encoder GPIO Pins

```dts
// ❗ REPLACE with your actual encoder connections:
a-gpios = <&xiao_d 4 (GPIO_ACTIVE_HIGH | GPIO_PULL_UP)>;  // Encoder A
b-gpios = <&xiao_d 5 (GPIO_ACTIVE_HIGH | GPIO_PULL_UP)>;  // Encoder B
input-gpios = <&xiao_d 6 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>; // Encoder switch

// ❗ ADJUST: Based on your encoder resolution
steps = <80>; // Typical: 20, 24, 30, 80
```

### 🌈 **RGB LED Configuration** (id1.overlay)

#### 5. Main RGB Strip (SPI)

```dts
// ❗ VERIFY: SPI pin assignment for main keyboard LEDs
&xiao_spi {
    // Check if this uses correct SPI pins for your setup
}

led_strip_main: ws2812@0 {
    chain-length = <64>; // ❗ UPDATE: Your exact main LED count
}
```

#### 6. Numpad RGB Strip (GPIO)

```dts
// ❗ REPLACE with your actual numpad RGB data pin:
gpios = <&xiao_d 7 0>;
chain-length = <18>; // ❗ UPDATE: Your exact numpad LED count
```

### 🖥️ **OLED Display** (id1.overlay)

#### 7. OLED I2C Address

```dts
oled: ssd1306@3c {
    reg = <0x3c>; // ❗ VERIFY: Common addresses are 0x3c or 0x3d
}
```

### 🗺️ **Matrix Layout** (id1-layouts.dtsi)

#### 8. Physical Key Layout

```dts
// ❗ CRITICAL: Map all 64 main keys to correct physical positions
// ❗ CRITICAL: Map all 18 numpad keys to correct positions
// Current layout is PLACEHOLDER - update with your actual layout
```

### ⌨️ **Keymap Configuration** (id1.keymap)

#### 9. Key Assignments

```dts
// ❗ CUSTOMIZE: Update keymap for your preferred layout
// ❗ VERIFY: Encoder bindings match your usage preferences
// ❗ ADD: Custom RGB and OLED controls as needed
```

### ⚙️ **Feature Configuration** (id1.conf)

#### 10. RGB and Power Settings

```conf
# ❗ ADJUST: If using external power for RGB
CONFIG_ZMK_RGB_UNDERGLOW_EXT_POWER=y

# ❗ TUNE: RGB effects and brightness limits
CONFIG_ZMK_RGB_UNDERGLOW_BRT_MIN=5
CONFIG_ZMK_RGB_UNDERGLOW_BRT_MAX=100
```

#### 11. OLED Settings

```conf
# ❗ VERIFY: OLED dimensions match your display
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_WIDGET_LAYER_STATUS=y
CONFIG_ZMK_WIDGET_WPM_STATUS=y
```

## 🚨 **MAJOR ISSUE: Custom Driver Required**

### 12. PCF8575 Matrix Driver Implementation

**❗ CRITICAL PROBLEM**: The current configuration **WILL NOT WORK** because:

```dts
// ❌ THIS IS WRONG - ZMK doesn't support PCF8575 for kscan:
kscan_main: kscan_main {
    compatible = "zmk,kscan-gpio-matrix"; // This won't work with I2C expanders
}
```

**Required Solutions**:

#### Option A: Implement Custom PCF8575 Driver

1. Create custom kscan driver in ZMK
2. Implement I2C communication to PCF8575
3. Handle matrix scanning via I2C
4. **Estimated effort**: 2-4 weeks for experienced developer

#### Option B: Use Direct GPIO Connections

1. Remove PCF8575 expanders entirely
2. Connect each key directly to nRF52840 GPIO pins
3. Update configuration to use `zmk,kscan-gpio-matrix`
4. **Limitation**: nRF52840 has limited GPIO pins

#### Option C: Switch to QMK

1. Use the provided QMK configuration instead
2. Benefit from proven I2C expander support
3. **Estimated effort**: 1-2 days to get working

## 🧪 **Testing and Validation**

### 13. I2C Communication Test

```bash
# Use nRF Connect or similar tool to scan I2C bus
# Verify PCF8575 devices respond at correct addresses
```

### 14. Matrix Testing

```bash
# After implementing custom driver:
# - Test each key individually
# - Verify no ghosting or chattering
# - Check matrix scanning performance
```

## 🛠️ **Development Workflow**

### Building ZMK

```bash
# Install west build tool
pip install west

# Initialize ZMK workspace
west init -l zmk-config/
west update

# Build firmware
west build -p -b seeeduino_xiao_ble -- -DSHIELD=id1
```

### Flashing to Device

```bash
# Copy UF2 file to bootloader drive
cp build/zephyr/zmk.uf2 /path/to/XIAO-SENSE/
```

## 📊 **Effort Comparison**

| Approach                | Time Estimate | Difficulty | Success Rate |
| ----------------------- | ------------- | ---------- | ------------ |
| **QMK (Recommended)**   | 1-2 days      | Medium     | 95%          |
| **ZMK + Custom Driver** | 2-4 weeks     | Very High  | 60%          |
| **ZMK + Direct GPIO**   | 1 week        | High       | 80%          |

## 🚀 **Recommendation**

**Switch to QMK firmware** for this build. ZMK is excellent for simple keyboards or when Bluetooth is critical, but your I2C expander setup is much better supported in QMK.

### Why QMK is Better for This Project:

- ✅ Proven PCF8575/I2C expander support
- ✅ Multiple working examples in community
- ✅ Better debugging tools and documentation
- ✅ VIA support for easy keymap editing
- ✅ Faster development and troubleshooting

### When to Use ZMK:

- 🔋 Battery-powered keyboards requiring low power
- 📱 Bluetooth-first usage scenarios
- 🎯 Simple matrix keyboards without I2C expanders
- 🏗️ Learning modern embedded development

## 📚 **ZMK Resources**

- [ZMK Documentation](https://zmk.dev/docs)
- [Custom Shield Development](https://zmk.dev/docs/development/new-shield)
- [ZMK Discord Community](https://discord.gg/8cfMkQksSB)
- [Zephyr RTOS I2C Documentation](https://docs.zephyrproject.org/latest/hardware/peripherals/i2c.html)

---

## 🏁 **Success Criteria (If You Continue with ZMK)**

✅ **You'll know it's working when**:

- Custom PCF8575 driver successfully implemented
- All 64 main keys register through I2C expander
- All 18 numpad keys register through I2C expander
- RGB lighting and OLED display functional
- Bluetooth connectivity established
- Power consumption optimized for battery use

**⚠️ Realistic Assessment**: Unless you're specifically learning ZMK development or require Bluetooth, **the QMK option will be significantly faster and more reliable** for getting a working keyboard.
