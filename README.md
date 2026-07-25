# ZMK PAW3222 Driver

This driver enables the use of the PixArt PAW3222 optical sensor with the ZMK framework.
To support its use in KobitoKey, this driver includes sensor angle correction.
X/Y movement data from the PAW3222 can be rotated by a configurable angle before being passed to ZMK's input processing system.


## Overview

The PAW3222 is a low-power optical mouse sensor suitable for tracking applications such as mice and trackballs. This driver communicates with the PAW3222 sensor via SPI interface.

## Installation

1. Add as a ZMK module in your west.yml:

```
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
    - name: juichi50iii
      url-base: https://github.com/juichi50iii

  projects:
    - name: zmk
      remote: zmkfirmware
      revision: main
      import: app/west.yml
    - name: zmk-driver-paw3222
      remote: juichi50iii
      revision: main
```

## Device Tree Configuration

Configure in your shield or board config file (.overlay or .dtsi):

```dts
&pinctrl {
    spi0_default: spi0_default {
        group1 {
            psels = <NRF_PSEL(SPIM_SCK, 0, 5)>,
                <NRF_PSEL(SPIM_MOSI, 0, 4)>,
                <NRF_PSEL(SPIM_MISO, 0, 4)>;
        };
    };

    spi0_sleep: spi0_sleep {
        group1 {
            psels = <NRF_PSEL(SPIM_SCK, 0, 5)>,
                <NRF_PSEL(SPIM_MOSI, 0, 4)>,
                <NRF_PSEL(SPIM_MISO, 0, 4)>;
            low-power-enable;
        };
    };
};

&spi0 {
    status = "okay";
    compatible = "nordic,nrf-spim";
    pinctrl-0 = <&spi0_default>;
    pinctrl-1 = <&spi0_sleep>;
    pinctrl-names = "default", "sleep";
    cs-gpios = <&gpio0 13 GPIO_ACTIVE_LOW>;

    trackball: trackball@0 {
        status = "okay";
        compatible = "pixart,paw3222";
        reg = <0>;
        spi-max-frequency = <2000000>;
        irq-gpios = <&gpio0 2 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;
        rotation-angle = <25>;
    };
};
```

## Enable the module in your keyboard's Kconfig file

Add the following to your keyboard's `Kconfig.defconfig`:

```kconfig
if ZMK_KEYBOARD_YOUR_KEYBOARD

config ZMK_POINTING
    default y

config PAW3222
    default y

endif
```

## Properties

- `irq-gpios`: GPIO connected to the motion pin (required)
- `res-cpi`: CPI resolution for the sensor (optional)
- `force-awake`: Initialize the sensor in "force awake" mode (optional, boolean)
- `rotation-angle`: Rotation angle applied to the reported X/Y movement (optional,integer)

Examples:
/* No rotation */
rotation-angle = <0>;

/* 25-degree rotation */
rotation-angle = <25>;

/* 25-degree rotation in the opposite direction */
rotation-angle = <(-25)>;

---

# ZMK PAW3222 ドライバ

このドライバは、PixArt PAW3222光学センサーをZMKフレームワークで使用できるようにします。
小人キーでの使用に際してセンサー角度の補正にも対応した次第です。
PAW3222から取得したX/Y移動量を、ZMKの入力処理へ渡す前に任意の角度で回転できます。

## 概要

PAW3222は、マウスやトラックボールなどのトラッキングアプリケーションに適した低消費電力の光学マウスセンサーです。このドライバはSPIインターフェースを介してPAW3222センサーと通信します。

## インストール

1. ZMKモジュールとして追加：

```
# west.yml に追加
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
    - name: juichi50iii
      url-base: https://github.com/juichi50iii
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: main
      import: app/west.yml
    - name: zmk-driver-paw3222
      remote: juichi50iii
      revision: main
```

## デバイスツリー設定

シールドまたはボード設定ファイル（.overlayまたは.dtsi）で設定：

```dts
&pinctrl {
    spi0_default: spi0_default {
        group1 {
            psels = <NRF_PSEL(SPIM_SCK, 0, 5)>,
                <NRF_PSEL(SPIM_MOSI, 0, 4)>,
                <NRF_PSEL(SPIM_MISO, 0, 4)>;
        };
    };

    spi0_sleep: spi0_sleep {
        group1 {
            psels = <NRF_PSEL(SPIM_SCK, 0, 5)>,
                <NRF_PSEL(SPIM_MOSI, 0, 4)>,
                <NRF_PSEL(SPIM_MISO, 0, 4)>;
            low-power-enable;
        };
    };
};

&spi0 {
    status = "okay";
    compatible = "nordic,nrf-spim";
    pinctrl-0 = <&spi0_default>;
    pinctrl-1 = <&spi0_sleep>;
    pinctrl-names = "default", "sleep";
    cs-gpios = <&gpio0 13 GPIO_ACTIVE_LOW>;

    trackball: trackball@0 {
        status = "okay";
        compatible = "pixart,paw3222";
        reg = <0>;
        spi-max-frequency = <2000000>;
        irq-gpios = <&gpio0 2 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;
        rotation-angle = <25>;
    };
};
```

## キーボードのKconfigファイルでモジュールを有効化

キーボードの `Kconfig.defconfig` に以下を追加：

```kconfig
if ZMK_KEYBOARD_YOUR_KEYBOARD

config ZMK_POINTING
    default y

config PAW3222
    default y

endif
```

## プロパティ

- `irq-gpios`: モーションピンに接続されたGPIO（必須）
- `res-cpi`: センサーのCPI解像度（任意）
- `force-awake`: センサーを「強制起動」モードで初期化（任意、ブール値）
- `rotation-angle`: X/Y移動量に適用する回転角度 (任意)

例：
/* 回転なし */
rotation-angle = <0>;

/* 25度回転 */
rotation-angle = <25>;

/* 反対方向へ25度回転 */
rotation-angle = <(-25)>;