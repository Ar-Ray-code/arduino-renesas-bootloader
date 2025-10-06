# Arduino Renesas ブートローダー

このリポジトリでは、EK-RA6M5をPortenta C33（RA6M5）として使用するためのArduino RenesasブートローダーのビルドとJTAG書き込み手順を説明します。

## 概要
- 対象ボード: EK-RA6M5

## J-Link のインストール（Ubuntu 22.04）
- SEGGER 公式の .deb パッケージを使用します。以下はライセンス同意を含んだダウンロード手順です。

```
cd /tmp
curl -L -o JLink_Linux_x86_64.deb \
  -H "Referer: https://www.segger.com/downloads/jlink/" \
  -H "Origin: https://www.segger.com" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data "accept_license_agreement=accepted&non_emb_ctr=confirmed" \
  https://www.segger.com/downloads/jlink/JLink_Linux_x86_64.deb

sudo apt install ./JLink_Linux_x86_64.deb

sudo udevadm control --reload-rules
sudo udevadm trigger

# バージョン確認
JLinkExe -NoGui 1 || true
```

### 注意
- 初回はUSBの再挿抜、もしくはホスト再起動が必要な場合があります。
- `plugdev` グループへの所属が必要です。

## ブートローダーのビルドと書き込み

EK-RA6M5のDEBUG1 (J10) のUSBと書き込み用PCのUSBを接続します。

![IMG_9808](https://github.com/user-attachments/assets/f00d3726-7cd3-4eca-809b-55211ecd3e80)

```bash
# Clone TinyUSB, arduino-renesas-bootloader
git clone https://github.com/hathach/tinyusb.git
git clone https://github.com/Ar-Ray-code/arduino-renesas-bootloader.git

# Apply patch to TinyUSB
cd tinyusb
git checkout 0.17.0
python3 ./tools/get_deps.py ra
export TINYUSB_ROOT=$PWD
patch -p1 < ../arduino-renesas-bootloader/0001-fix-arduino-bootloaders.patch

# back to arduino-renesas-bootloader
cd ../arduino-renesas-bootloader

./compile.sh
```

JFlashLiteで書き込みます。

```bash
JFlashLite
```



### PlatformIO での利用

- DFUモードでのUSB接続に必要です。[99-arduino-dfu.rules](rules/99-arduino-dfu.rules)を`/etc/udev/rules.d/` に配置し、`sudo udevadm control --reload-rules && sudo udevadm trigger` を実行してください。

`platformio.ini` の例: 

```ini
[env:portenta_c33]
platform = platformio/renesas-ra@^1.7.0
board = portenta_c33

; change microcontroller
board_build.mcu = r7fa6m5bh2cbg

; change MCU frequency
board_build.f_cpu = 200000000L

framework = arduino
```

ソースコードは[Portenta C33 User Manual
](https://docs.arduino.cc/tutorials/portenta-c33/user-manual)を参考にしてください。

書き込みは、EK-RA6M5のUSB HIGH SPEED (J31) のUSBと書き込み用PCのUSBを接続します。

![IMG_9809](https://github.com/user-attachments/assets/795ec467-393e-45db-809a-93fabcc94d85)


<br>

## (参考) TinyUSB の取得とパッチ適用手順

```
git clone https://github.com/arduino/arduino-renesas-bootloader
git clone https://github.com/hathach/tinyusb.git
cd tinyusb
git checkout 0.17.0
python ./tools/get_deps.py ra
export TINYUSB_ROOT=$PWD
patch -p1 < ../arduino-renesas-bootloader/0001-fix-arduino-bootloaders.patch
cd ..
cd arduino-renesas-bootloader
./compile.sh
```
