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

JFlashLiteを起動すると、次のような画面が出ます。OKを押します。

<img width="485" height="374" alt="Screenshot from 2025-10-06 22-23-30" src="https://github.com/user-attachments/assets/d070c7eb-56b2-4f17-94f1-1f1340b87e43" />

デバイスの特定に失敗した場合は、 "Target Device" の横の "…" をクリックして、対象となるボードを選択します。

<img width="600" height="499" alt="Screenshot from 2025-10-06 22-23-23" src="https://github.com/user-attachments/assets/adff0c5a-88f6-4a19-8d89-e3fb52c3e60a" />

"Data File" を選択できるようになるので、生成されたhex（ここでは "dfu_c33.hex"）を選択します。

<img width="549" height="661" alt="image" src="https://github.com/user-attachments/assets/0b31e2ff-436f-4c90-850a-7bb8816aa08f" />

"Program Device" を押して書き込みます。 

<img width="559" height="675" alt="image" src="https://github.com/user-attachments/assets/9467ad3b-1992-4365-8d7e-69ccb907470d" />

"Done." と出たら終了です。画面を閉じます。

### PlatformIO での利用

- DFUモードでのUSB接続に必要です。[99-arduino-dfu.rules](rules/99-arduino-dfu.rules)を`/etc/udev/rules.d/` に配置し、`sudo udevadm control --reload-rules && sudo udevadm trigger` を実行してください。

`lsusb` で確認すると、 `2341:0368 Arduino SA Portenta C33 DFU` と出てきます。

<img width="800" height="155" alt="image" src="https://github.com/user-attachments/assets/34cb08f6-033a-4ec9-8639-2ed34434fb51" />

platformio.iniの例: 

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
