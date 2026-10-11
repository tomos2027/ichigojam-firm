# IchigoJam firm

## 構成 / structure

- `IchigoJam_BASIC/` IchigoJam BASIC source code
- `IchigoJam_P` IchigoJam BASIC for Raspberry Pi Pico
- `console` IchigoJam BASIC for console (beta)
- `SPRESENSE` IchigoJam BASIC for SPRESENSE (beta)
- `IchigoJam_MCX` IchigoJam BASIC for NXP MCX-A153 (beta)

## ライセンス / license

### IchigoJam本体 / IchigoJam core

Copyright 2014-2025 the IchigoJam authors. All rights reserved. MIT license.

### 4K版拡張 / 4K version extensions (GivetakeJam_P準拠 / based on GivetakeJam_P)

4K版の拡張機能 (プログラム領域4096バイト化、拡張配列変数、`IR.IN`、`ENV.IN`等) は [GivetakeJam_P](https://github.com/Canon-boy4/ichigojam-firm) を参考にした実装であり、この拡張分は GivetakeJam_P と同じライセンス (MIT license, Copyright 2014-2025 the IchigoJam authors) に従います。
The 4K version extensions (4096-byte program area, extended array variables, `IR.IN`, `ENV.IN`, etc.) are implemented with reference to [GivetakeJam_P](https://github.com/Canon-boy4/ichigojam-firm), and these extensions follow the same license as GivetakeJam_P (MIT license, Copyright 2014-2025 the IchigoJam authors).

### NTSC出力コード / NTSC output code (`IchigoJam_P/src/ntsc/`)

NTSCコンポジット映像出力コード (`IchigoJam_P/src/ntsc/`) は KenKen 氏による著作物です。
The NTSC composite video output code (`IchigoJam_P/src/ntsc/`) is copyrighted by KenKen.

KenKen氏のライセンス（原文 / original text）:

> 当サイトに掲載の私が作成した回路やソフトウェアについては、営利目的でなければ、特に個別に記載のない限り自由にご利用いただけます。利用したものを公開する場合は、利用している旨をどこか（Webサイト等）に記載いただけますと幸いです。なお、ご利用によって生じた被害や損害については責任を負いかねます。
> 営利利用については、個別判断いたしますので、掲示板等でご連絡ください。

English (license notice from the source code header):

> This program supplied herewith by KenKen is free software; you can redistribute it and/or modify it under the terms of the same license written here and only for non-commercial purpose. The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software. This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of FITNESS FOR A PARTICULAR PURPOSE. The copyright owner and contributors are NOT LIABLE for any damages caused by using this program.

- 著作者 / Author: [KenKen](http://www.ze.em-net.ne.jp/~kenken/profile/index.html)
- Copyright (C) 2024, KenKen, all right reserved.

### 利用時の注意 / Notice

- このコードを利用する場合は、それぞれのライセンスを確認して、それぞれのライセンスに従ってください。
  (When using this code, please check each license and follow each respective license.)
- 本リポジトリの内容は、いかなる保証も行うものではありません。利用によって生じたいかなる損害・不利益についても、一切の責任を負いません。
  (The contents of this repository are provided "AS IS", without warranty of any kind. The authors are not liable for any damages or disadvantages arising from the use of them.)

## 商標 / trademark

- IchigoJamは、株式会社jig.jpの登録商標です  (IchigoJam is a registered trademark of jig.jp co., ltd.)
- SPRESENSEは、ソニーグループ（株）またはその関連会社の登録商標または商標です  (SPRESENSE is a registered trademark of SONY Group Corporation)
- Raspberry Piは、Raspberry Pi財団の登録商標です  (Raspberry Pi is a registered trademark of Raspberry Pi Foundation)
