# minichlink

CH32V003用のCH-LinkE $4プログラミングドングルを使用するための無料のオープンな仕組み。

## Linuxでのセットアップ

Linuxを使用する場合は、提供されているudevルールを `/etc/udev/rules.d/` にインストールしてください。
rootとして `make install_udev_rules` を実行することでインストールできます。このmakefileターゲットは、udevデーモンに新しいルールをリロードさせ、既に接続されているデバイスを再評価するため、再起動や再接続を省略できます。

## Windowsでのセットアップ

Windowsでは、必要に応じてWCHインターフェース1にWinUSBドライバーをインストールできます。

## システム要件

ここでの実行ファイルは約12kBで、libusbドライバーを除くすべてを含んでいます。Linuxでは `libusb-1.0-dev` が必要です。

## UIAPduino Pro Micro CH32V003 V1.4 macOSサポート

以下の手順でhomebrewを使用してRISC-Vツールチェーンをインストールしてください：

- [RISC-V Homebrew Repository](https://github.com/riscv-software-src/homebrew-riscv)

### 修正されたファイル

- Makefile:

```makefile
CFLAGS := $(ARCHFLAG) -O0 -Wall -Wno-asm-operand-widths -Wno-deprecated-declarations -Wno-deprecated-non-prototype -D__MACOSX__ -DMINICHLINK -DCH32V003 -I. $(LIBUSB_INCS) -DDEFAULT_CHLINK_PID=0xb803
```

- minichlink.c:

UIAPduino Pro Micro CH32V003 V1.4でCH-Link PID 0xb803をサポートするためにコードを変更

55行目:
```c
// 元のコード
else if( strcmp( specpgm, "b003boot" ) == 0 )
    dev = TryInit_B003Fun(SimpleReadNumberInt(init_hints->serial_port, 0x1209b003));

// UIAPduino Pro Micro CH32V003 V1.4用にCH-Link PID 0xb803で修正
else if( strcmp( specpgm, "b003boot" ) == 0 )
    dev = TryInit_B003Fun(SimpleReadNumberInt(init_hints->serial_port, 0x1209b803));
```

74行目:

```c
// 元のコード
else if ((dev = TryInit_B003Fun(SimpleReadNumberInt(init_hints->serial_port, 0x1209b003))))
{
    fprintf( stderr, "Found B003Fun Bootloader\n" );
}

// UIAPduino Pro Micro CH32V003 V1.4用にCH-Link PID 0xb803で修正
else if ((dev = TryInit_B003Fun(SimpleReadNumberInt(init_hints->serial_port, 0x1209b803))))
{
    fprintf( stderr, "Found B003Fun Bootloader\n" );
}
```

### 注意事項

- minichlinkをArduino IDEの「tools」フォルダにコピーしてください：
  `/Arduino/Arduino15/packages/UIAP/tools/minichlink-2982dfd/1.0.0/`

### 主な改善点

1. **CH-Linkサポートの強化**: `DEFAULT_CHLINK_PID=0xb803` の定義により、macOSでのCH-Linkプログラミングドングルとの互換性が向上
2. **警告の抑制**: macOS固有のコンパイラ警告を抑制し、クリーンなビルドを実現
3. **ユニバーサルバイナリサポート**: ARCHフラグにより特定のアーキテクチャ（x86_64/arm64）用のビルドが可能

## 使用方法

```
Usage: minichlink [args]
 single-letter args may be combined, i.e. -3r
 multi-part args cannot.
 -3 Enable 3.3V                    3.3Vを有効化
 -5 Enable 5V                     5Vを有効化
 -t Disable 3.3V                  3.3Vを無効化
 -f Disable 5V                    5Vを無効化
 -u Clear all code flash - by power off (also can unbrick) すべてのコードフラッシュをクリア（電源オフ）、復旧にも使用
 -b Reboot out of Halt             ハルト状態から再起動
 -e Resume from halt               ハルト状態から再開
 -a Place into Halt                ハルト状態に移行
 -D Configure NRST as GPIO         NRSTをGPIOとして設定
 -d Configure NRST as NRST        NRSTをNRSTとして設定
 -s [debug register] [value]      デバッグレジスタに値を設定
 -g [debug register]               デバッグレジスタの値を取得
 -w [binary image to write] [address, decimal or 0x, try0x08000000] バイナリイメージを書き込み
 -r [output binary image] [memory address, decimal or 0x, try 0x08000000] [size, decimal or 0x, try 16384] バイナリイメージを読み出し
   Note: for memory addresses, you can use 'flash' 'launcher' 'bootloader' 'option' 'ram' and say "ram+0x10" for instance
   注意：メモリアドレスには 'flash' 'launcher' 'bootloader' 'option' 'ram' が使用でき、"ram+0x10" のように指定可能
   For filename, you can use - for raw or + for hex.
   ファイル名には - をrawデータに、+ をhexデータに使用できます
 -T is a terminal. This MUST be the last argument.
   -Tはターミナルモード。これが最後の引数である必要があります
```
