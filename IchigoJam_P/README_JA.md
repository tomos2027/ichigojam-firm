# IchigoJam P

## 環境構築
- **CMake**と**GCC**をインストールする  

- 以下のライブラリを`IchigoJam_P`ディレクトリ直下に置く  
    - **IchigoJam_BASIC**: 1つ上の階層にある`IchigoJam_BASIC`ディレクトリをコピーする  
    - **pico-sdk**: https://github.com/raspberrypi/pico-sdk.git 最新版 (2.3.1、コミット `079c6f3`) `git submodule update --init --recursive` でサブモジュールも更新する  
    - **pico-extras**: https://github.com/raspberrypi/pico-extras.git master (コミット `52fd7a7`)  
    - **PicoDVI**: https://github.com/mlorenzati/PicoDVI.git master (動作確認: コミット `9eca921`)  

- PicoDVIパッチを当てる (SWITCHコマンドによるDVI/NTSC切替の安定化に必須)  
    `PicoDVI`ディレクトリで`patches/picodvi_dvi_stop_race_fix.patch`を適用する  
    ```sh
    cd PicoDVI  
    git apply ../patches/picodvi_dvi_stop_race_fix.patch  
    ```
    - `dvi_stop()`でDVI出力を止めるとき、DMA割り込みハンドラが停止済みチャンネルの完了待ちで固まる競合を修正する (停止中に残った割り込みを無視する)  

- TinyUSBパッチを当てる (USBキーボード利用に必須)  
    `pico-sdk/lib/tinyusb/src/osal/osal_pico.h`の`osal_task_delay()`をbusy loopに置き換える  
    ```c
    TU_ATTR_ALWAYS_INLINE static inline void osal_task_delay(uint32_t msec) {
        // sleep_ms(msec); // IchigoJam Pではタイマー割り込み内から呼ばれる
        uint64_t start = time_us_64();
        while (time_us_64() - start < msec * 1000) {
            ;
        }
    }
    ```

- 以下のパスを通す  
    ```sh
    PICO_SDK_PATH=foo/IchigoJam_P/pico-sdk  
    PICO_EXTRAS_PATH=foo/IchigoJam_P/pico-extras  
    ```

詳細は以下を参照  
[Raspberry Pi Picoをセットアップしよう](https://datasheets.raspberrypi.com/pico/getting-started-with-pico-JP.pdf)  

## ビルド
```sh
cd IchigoJam_P  
mkdir build  
cd build  
cmake ..  
make  
```

`IchigoJam_P.uf2`ファイルが作成できたらpicoに書き込む  

## 4K版の拡張機能 (GivetakeJam_P準拠)
- **プログラム領域を1024バイト → 4096バイトに拡張** (LISTは`#E00`へ移動、`FREE`で残り容量を確認できる)
- **配列変数を拡張**： `[0]`〜`[101]`に加えて`[102]`〜`[357]`が使える (`#C00`-#DFFのVAR2領域)
    - `FOR`/`NEXT`、`LET`、`INPUT`でも拡張配列変数が使える
- **保存本数**： 内蔵flashに25本 (1本4096バイト化に伴う変更、`FILES`で0〜24を表示)
- **外部EEPROM**： ファイル番号100〜131 (1スロット4096バイト)。24LC64/24LC256/24FC1025に単一バイナリで対応、32バイトページ書き込みで安全性向上
    - `FILES0`でEEPROMの有無を自動判定して表示する
- **`WS.LED port,n[,repeat]`** WS2812B LEDテープ制御 (ポート選択式)
    - port: 1-7 (OUT1-6、7=LED/GP2)、n: LED数、repeat: 送信回数
    - 色データは配列変数`[0]`〜から読み込む (1LED=3変数、拡張配列に対応し**最大119LED**)
    - DVI (252MHz) / NTSC (157.5MHz) 両モードで正しいタイミングで送信する
- **`IR.IN port,[n]`** NEC方式赤外線受信 (HX1838系モジュールをIN1-4に接続)
    - [n+0]〜[n+3]=受信コード、[n+4]=リピートフラグ、[n+5]=エラーコード、[n+6]=モード (0=NEC標準/1=NEC拡張)
- **`ENV.IN [n]`** 環境センサー読み取り (AHT20+BMP280をI2Cに接続)
    - [n+0]=温度0.1℃、[n+1]=湿度0.1%RH、[n+2]=気圧0.1hPa、[n+3]=BMP280温度0.1℃、[n+4]=エラー、[n+5]=状態フラグ (AHT20が正常ならBMP280未接続でも動作)

### 注意
- LIST領域が`#C00`から`#E00`へ移動したため、`POKE #C00`系でRAM直アクセスする既存プログラムは動作が変わります
- 4K版の実装は [GivetakeJam_P](https://github.com/Canon-boy4/ichigojam-firm) (MIT license, Copyright 2014-2025 the IchigoJam authors) を参考にしています

## 映像出力
- **DVI (HDMI)**と**NTSCコンポジット**に対応。SWITCHコマンド (F10キー) で切り替え  
    - `SWITCH 0` ... DVI (HDMI)  
    - `SWITCH 1` ... NTSCコンポジット  
    - `SWITCH` ... トグル  
- 選択したモードはflashに保存され、次回起動時にそのモードで起動する  
- NTSC画面は32x24文字で画面いっぱいに表示される  
- NTSC信号は**GP3** (5番ピン) から出力される。下記の回路を介して映像入力に接続する  
    ```
    GP3 -- 75Ω --- 映像出力
          |
         === 560pF
          |
         GND
    ```
- NTSCモードはCPUが157.5MHz駆動のため、DVIモード (252MHz) よりBASICの実行速度は遅くなる (オリジナルのIchigoJamよりは速い)  
- NTSCモードでは基板のLEDが約1Hzで点滅する (動作表示)  
    - 高速点滅 ... 内部異常を検知した  
    - 点滅が止まったまま ... 停止している。約8秒後にウォッチドッグが自動で再起動する  
- 電源を切る際は**5秒以上**待ってから再投入する (RP2040の仕様で、短い電源断では再起動せず停止したままになることがある。この場合も約8秒で自動復帰する)  

## 動作がおかしくなったら
`build`ディレクトリを消して作り直してみる  
特に映像の出力が正常に行われなくなることが多い  

画面に「E+番号」が表示されたら内部状態の異常を検知している (1:フォントポインタ 2:VRAMポインタ 4:ビデオモード 8:フォントRAM境界 16:DMAバッファ境界 32:走査線カウンタ)  

## ライセンス / license
- Copyright 2014-2025 the IchigoJam authors. All rights reserved. MIT license. https://ichigojam.net/license

## 商標 / Trademark
IchigoJamは、株式会社jig.jpの登録商標です(IchigoJam is a registered trademark of jig.jp co., ltd.)  
Raspberry Pi は、Raspberry Pi財団の登録商標です(Raspberry Pi is a registered trademark of the Raspberry Pi Foundation)  

## 利用規約に同意の上、ダウンロードしてください。第三者への配布にはライセンスが必要になります。詳しくはこちら。https://pcn.club/sp/ijp/
- 利用規約（IchigoJamロイヤリティフリープログラム利用規約） https://ichigojam.net/ichigojam-license.pdf

株式会社ナチュラルスタイル  
2024/12/25 作成
