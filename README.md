# Arduino Renesas ブートローダー（C33/JTAG対応、日本語版）

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

sudo dpkg -i ./JLink_Linux_x86_64.deb || sudo apt-get -y -f install

# udev ルールの反映（通常は自動で実行されますが、念のため）
sudo udevadm control --reload-rules
sudo udevadm trigger

# バージョン確認
JLinkExe -NoGui 1 || true

# 接続確認（接続中のエミュレータ一覧を表示）
printf "ShowEmuList\nq\n" | JLinkExe -NoGui 1 -CommandFile /dev/stdin
```

### 注意
- インストールにより `/usr/bin/JLinkExe` 等のシンボリックリンクが作成されます。
- 初回はUSBの再挿抜、もしくはホスト再起動が必要な場合があります。
- 権限周りの問題がある場合は、現在のユーザーが `plugdev` 等に属しているかをご確認ください。

## ブートローダーのビルドと書き込み
### 事前準備（TinyUSB の取得とパッチ適用）
1. リポジトリ取得
   - `git clone https://github.com/arduino/arduino-renesas-bootloader`
   - `git clone https://github.com/hathach/tinyusb.git`
2. TinyUSB のセットアップ
   - `cd tinyusb`
   - `git checkout 0.17.0`
   - `python3 ./tools/get_deps.py ra`
   - `export TINYUSB_ROOT=$PWD`
   - `patch -p1 < ../arduino-renesas-bootloader/0001-fix-arduino-bootloaders.patch`
3. 本リポジトリに戻る
   - `cd ../arduino-renesas-bootloaders`（あるいは適切なパス）

### ビルド（Portenta C33として）
- 単体ビルド（C33のみ）
  - `make -f Makefile.c33 -j$(nproc)`
  - 生成物: `_build/portenta_c33/arduino-renesas-bootloader.hex` および `.bin`
- 一括ビルド（配布用）
  - `./compile.sh`
  - `distrib/dfu_c33.hex` などが生成されます

### JTAG書き込み（J-Link）と検証
本リポジトリの `Makefile.c33` に J-Link 用のターゲットを追加しました。

EK-RA6M5のDEBUG1 (J10) のUSBと書き込み用PCのUSBを接続します。

![IMG_9808](https://github.com/user-attachments/assets/f00d3726-7cd3-4eca-809b-55211ecd3e80)

### 実行例
1. 接続確認（J-Linkに同梱の Commander でエミュレータ列挙）
   - `printf "ShowEmuList\nq\n" | JLinkExe -NoGui 1 -CommandFile /dev/stdin`
2. 書き込み＋検証（OB-S124 の例。SNは適宜置き換えてください）
   - `make -f Makefile.c33 JLINK_DEVICE=R7FA6M5BH JLINK_SN=831782252 flash_verify`
   - 先頭アドレス `0x00000000` に書き込み、書き込み後にフラッシュ先頭を `savebin` で読み出し、ローカルの `.bin` とバイト比較します。

### PlatformIO での利用

- DFUモードでのUSB接続に必要です。`/etc/udev/rules.d/` に配置し、`sudo udevadm control --reload-rules && sudo udevadm trigger` を実行してください。

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
