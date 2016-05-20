# Arduino I2Cライブラリ Wire改良版　V1.0

このライブラリは、**Arduino IDE 1.6.8**に付属するAVR版 I2Cライブラリ**Wire**を改良したものです。


## 改良点
* 受信バッファを32バイトから64バイトに変更
* スレーブモードで同時に複数のデバイスアドレスの実装対応  
  具体的にはtwi.cに下記の関数を追加しています。  
  * `void twi_set_addressMask(uint8_t msk)`  
  * `uint8_t twi_get_address(void)`  
  
  `twi_set_addressMask()`はI2Cスレーブアドレスマスクの設定します。  
  `twi_get_address()`は、データ受信時のターゲットI2Cスレーブアドレスの取得します。  
  
  

## 使用方法
### インストール
既存のWireライブラリを差し換えて下さい。

### 追加機能の無効化
差し換えた後に、追加修正機能を無効にしたい（バッファサイズも32バイトにする）場合は、  
*Wire\src\WIRE_EXPANSION.h*の下記の定義を変更して下さい。  
`#define WIRE_EXPANSION 1		// 1:機能拡張有効 0:無効`

### 複数のデバイスアドレスの実装
#### ヘッダファイルのインクルード
    #include <Wire.h>
    extern "C" {
    #include <utility/twi.h>
    }

#### Wireライブラリの初期設定
    void setup() {
        Wire.begin(0x50) ;                  // I2Cの初期化、自アドレスを0x50とする
        twi_set_addressMask(B0000100<<1);   // 複数アドレス対応 0x50|0x04=0x54もアドレスとする)
        Wire.onRequest(requestEvent) ;      // I2Cコマンド要求割込み関数の登録
        Wire.onReceive(receiveEvent) ;      // I2Cデータ受信割込み関数の登録
    }

twi_set_addressMask(uint8_t msk)関数でアドレスの比較時に指定したビットを比較しない設定を行うことが出来ます。  
mskには1ビット左シフトした値を設定します。0x54もアドレスとして利用するため、下位から3ビット目を1に設定(=0x04)した値を指定しています。

#### 受信時のターゲットアドレスの取得

    // I2Cマスタからのデータ受信ハンドラ
    void receiveEvent(int len) {
      uint16_t slaveAdr;                    // マスター要求スレーブアドレス
      slaveAdr = twi_get_address();         // ターゲットスレーブアドレス
    }

 `twi_get_address()`関数を使って、ターゲットとなるアドレスを取得します。
 
 

