# ==ASerialPacket クラスリファレンス==

ASerialPacket は、シリアル通信におけるパケットの構築・解析を担うクラスです。  
動作モードに応じて送受信対象が変化し、データ整合性・接続状態の管理・エラー処理などを内包しています。

---

## ==使用方法==
ASerialPacket クラスは、デバイスモードとコントローラモードの2つの動作モードを持ちます。
- **デバイスモード**：デバイス側で使用。デバイスIDとバージョンを指定して初期化します。
- **コントローラモード**：コントローラ側で使用。ターゲットデバイスIDを指定して初期化します。
動作モードに応じて、パケットの生成・解析方法が異なります。
---

## ==クラス概要==
| クラス名 | 説明 |
|----------|------|
| `ASerialPacket` | シリアル通信パケットの生成・解析を行うクラス |
| `ASerialDataStruct` | パケットデータを格納するための構造体群 |
| `ASerial::ErrorCodeList` | エラーコードを定義する列挙型 |
| `ASerialData` | 基本データ格納用の構造体 |
| `ToDeviceData` | コントローラからデバイスへのデータ構造体 |
| `ToControllerData` | デバイスからコントローラへのデータ構造体 |
| `ASerialDataStruct::ASerialData` | パケットデータの基本構造体 |
| `ASerialDataStruct::ToDeviceData` | コントローラからデバイスへのデータ構造体 |
| `ASerialDataStruct::ToControllerData` | デバイスからコントローラへのデータ構造体 |
----

## ==クラス定義==

```cpp
class ASerialPacket {
public:
  //===public定数===
  static const uint8_t MODE_DEVICE = 0;                   //デバイスモード
  static const uint8_t MODE_CONTROLLER = 1;               //コントローラモード
  static const uint8_t RESERVED_COMMAND_RESET = 0x00;     //予約済みコマンド リセット
  static const uint8_t RESERVED_COMMAND_GET_INFO = 0x01;  //予約済みコマンド デバイス情報リクエスト
  static const uint8_t DATA_NUM_MAX = _ASERIAL_MAX_DATA_NUM;                 //データ最大数
  //================

  
  /// @brief ASerialPacketコンストラクタ(deviceモード)
  /// @param _device_id device ID
  /// @param _device_ver device version
  ASerialPacket(uint8_t _device_id, uint8_t _device_ver);

  /// @brief ASerialPacketコンストラクタ(controllerモード)
  /// @param _target_device_id 
  ASerialPacket(uint8_t _target_device_id);


  /// @brief 動作モードを取得
  /// @return 動作モード(MODE_DEVICE or MODE_CONTROLLER)
  int GetMode(void);

  /// @brief IDを取得
  /// @return デバイスIDまたはターゲットデバイスID
  /// @note モードによって取得するIDが異なる。
  uint8_t GetID(void);

  /// @brief Verを取得
  /// @return デバイスVerまたは0(ControllerモードではVerは0)
  /// @note モードによって取得するVerが異なる。
  uint8_t GetVer(void);

  /// @brief 接続状態を取得
  /// @return 接続状態(true or false)
  /// @note コントローラモード時のみ有効。デバイスモードでは常にfalseを返す。
  bool GetConnectionState(void);

  /// @brief 最後に発生したエラーコードを取得
  /// @return 最後に発生したエラーコード
  /// @note エラーコードはASerial::ErrorCodeListに定義されています。
  uint16_t GetLastErrorCode(void);

   /// @brief 必要なパケットバッファサイズを取得(デバイスモード時)
  /// @param data_array データ配列 
  /// @param data_num データ数
  /// @return 必要なパケットバッファサイズ
  size_t GetNeedPacketBufSize(uint8_t *data_array, int data_num);

  /// @brief 必要なパケットバッファサイズを取得(コントローラモード時)
  /// @param command コマンド
  /// @param data_array データ配列 
  /// @param data_num データ数
  /// @return 必要なパケットバッファサイズ
  size_t GetNeedPacketBufSize(uint8_t command, uint8_t *data_array, int data_num);

  /// @brief 必要なパケットバッファサイズを取得(コントローラモード時)
  /// @param command コマンド
  /// @return 必要なパケットバッファサイズ
  size_t GetNeedPacketBufSize(uint8_t command);

  /// @brief 接続状態を設定
  /// @param state 接続状態
  void SetConnectionState(bool state);

protected:

  /// @brief パケットデータを読み取り
  /// @param _indata 受信したデータ(1バイト)
  /// @param data_buf_pt 受信したデータを格納するバッファへのポインタ
  /// @return 読み取り中は0、エラーが発生した場合は-1、読み取り完了で1を返す。
  int ReadPacketData(uint8_t _indata, ASerialDataStruct::ASerialData *data_buf_pt);


  /// @brief パケットデータを作成(コントローラモード時)
  /// @param to_device_data デバイスへ送信するデータ配列
  /// @param data_num データ数
  /// @param command 　コマンド
  /// @param data_packet_out パケットデータを格納するバッファへのポインタ
  /// @return 正常時は0、エラー時は-1を返す。
  /// @note コントローラモード時のみ使用可能。デバイスモードでは使用できません。
  /// @note 接続ステータスがfalseの場合、リセットコマンドやデバイス情報リクエスト以外のコマンドはエラーとなります。
  int MakePacketData(uint8_t *to_device_data, int data_num, uint8_t command, uint8_t *data_packet_out);

  /// @brief パケットデータを作成(コントローラモード時)*コマンドのみ
  /// @param command 　コマンド
  /// @param data_packet_out パケットデータを格納するバッファへのポインタ
  /// @return 正常時は0、エラー時は-1を返す。
  /// @note コントローラモード時のみ使用可能。デバイスモードでは使用できません。
  /// @note 接続ステータスがfalseの場合、リセットコマンドやデバイス情報リクエスト以外のコマンドはエラーとなります。
  int MakePacketData(uint8_t command, uint8_t *data_packet_out);


  /// @brief パケットデータを作成(デバイスモード時)
  /// @param to_controller_data コントローラへ送信するデータ配列
  /// @param data_num   データ数
  /// @param data_packet_out パケットデータを格納するバッファへのポインタ
  /// @return 正常時は0、エラー時は-1を返す。
  int MakePacketData(uint8_t *to_controller_data, int data_num, uint8_t *data_packet_out);

private:
  //===private定数===
  static const uint8_t AD_FLAG = 0xAD;      //加算フラグ値
  static const uint8_t DO_FLAG = 0xD0;      //スタートフラグ値
  //================
  uint8_t m_mode = MODE_DEVICE;   //動作モード設定
  uint8_t m_device_id = 0;        //デバイスID
  uint8_t m_device_ver = 0;       //デバイスVer
  uint8_t m_target_device_id = 0; //ターゲットデバイスID
  bool m_add_flag = false;        //加算フラグ
  bool m_read_packet = false;     //packet読み取りフラグ(true中はパケット読み取り中)
  ASerialDataStruct::ToDeviceData m_from_controller_data_buf;          //コントローラからのパケット読み取り中のデータバッファ
  ASerialDataStruct::ToControllerData m_from_device_data_buf;                  //デバイスからのパケット読み取り中のデータバッファ
  bool m_error_flag = false;        //何らかのエラーでパケットを無視するフラグ
  uint16_t m_lase_error_code = static_cast<uint16_t>(ASerial::ErrorCodeList::ERR_NONE);     //最後にエラーになったときのエラーコード
  bool m_connection_state = false;        //コントローラモード時の接続ステータス

  /// @brief フラグをリセット
  void ResetFlags(void);

  /// @brief データ配列をリセット
  void ResetDataArray(void);

  /// @brief データが加算フラグを必要とするかチェックし、必要ならば加算フラグを追加
  int CheckDataNeedAddFlag(uint8_t data, uint8_t *data_array, int *_index);
};
```

## ==コンストラクタ==

| 関数 | 説明 |
|------|------|
| `ASerialPacket(uint8_t device_id, uint8_t device_ver)` | デバイスモードで初期化 |
| `ASerialPacket(uint8_t target_device_id)` | コントローラモードで初期化 |

---

## ==動作モード定義==

| 定数名 | 値 | 説明 |
|--------|----|------|
| `MODE_DEVICE` | `0` | デバイスモード |
| `MODE_CONTROLLER` | `1` | コントローラモード |

---

## ==パブリック関数一覧==

| 関数 | 説明 |
|------|------|
| `int GetMode()` | 動作モードを取得 |
| `uint8_t GetID()` | モードに応じたIDを返す（Device または Target） |
| `uint8_t GetVer()` | デバイスバージョン（Controller時は0） |
| `bool GetConnectionState()` | 接続状態の取得（Controllerモードで有効） |
| `uint16_t GetLastErrorCode()` | 最後に発生したエラーコード（ASerial::ErrorCodeList） |
| `size_t GetNeedPacketBufSize(uint8_t* data_array, int data_num)` | 必要なバッファサイズの計算 |
| `void SetConnectionState(bool state)` | 接続状態の設定 |

---

## ==パケット操作関数（protected）==

| 関数 | 説明 |
|------|------|
| `int ReadPacketData(uint8_t indata, ASerialData* data_buf_pt)` | 受信バイトの逐次解析 |
| `int MakePacketData(...)` | パケット作成（モード・引数によりオーバーロード） |

> コントローラモードで使用時は接続状態が `false` の場合、`RESET` や `GET_INFO` 以外はエラーになります。

---

## ==内部データ構造体（`ASerialDataStruct` 名前空間）==

| 構造体名 | 概要 |
|----------|------|
| `ASerialData` | 基本データ格納（コマンド + データ配列） |
| `ToDeviceData` | コントローラ → デバイス用構造体 |
| `ToControllerData` | デバイス → コントローラ用構造体 |

---

## ==補助機能（private）==

- `ResetFlags()`：状態フラグのリセット  
- `ResetDataArray()`：内部バッファの初期化  
- `CheckDataNeedAddFlag()`：加算フラグの必要性チェック・処理  

---

## ==定数一覧==

| 名前 | 値 | 説明 |
|------|-----|------|
| `AD_FLAG` | `0xAD` | 加算フラグ用定数 |
| `DO_FLAG` | `0xD0` | スタートフラグ用定数 |
| `RESERVED_COMMAND_RESET` | `0x00` | リセットコマンド |
| `RESERVED_COMMAND_GET_INFO` | `0x01` | デバイス情報要求コマンド |
| `DATA_NUM_MAX` | `32` | データ配列の最大長 |

---

## ==エラーコード一覧==
| エラーコード | 説明 |
|--------------|------|
| `ASerial::ErrorCodeList::ERR_NONE` | 正常 |
| `ASerial::ErrorCodeList::WARNING_READ_SKIP` | パケットの読み飛ばし発生（警告） |
| `ASerial::ErrorCodeList::ERR_ADDFLAG_CASCADE` | 加算フラグ連続エラー |
| `ASerial::ErrorCodeList::ERR_DEVICE_ID_MISMATCH` | デバイスIDとターゲットデバイスIDの不一致 |
| `ASerial::ErrorCodeList::ERR_DATA_COUNT_INFO_OVER` | 送信データ数情報がデータ数上限を超過 |
| `ASerial::ErrorCodeList::ERR_CHECK_DATA_MISMATCH` | チェックデータの不整合 |
---


## ==使用例==

### データ送信パケット生成サンプル(Deviceモード) 

```cpp
// MIT License
// Copyright(c) 2025 NextAmusement
// See LICENSE file in the root directory.

// デバイスモードで初期化(ID:0x01 Ver:0x10);
ASerialPacket device(0x01, 0x10); 

const size_t DATA_NUM = 3;  //データ数
uint8_t sampleData[DATA_NUM] = {10, 20, 30}; //送信するデータ

//パケット格納バッファサイズの取得
int buf_size = device.GetNeedPacketBufSize(sampleData, DATA_NUM);

//パケット格納バッファの確保
uint8_t *packet_buf = new uint8_t[buf_size];

// packet_buf にパケットが格納される
device.MakePacketData(sampleData, DATA_NUM, packet_buf);

//~~~packet_bufを環境ごとの方法で送信する~~~

//確保したバッファの開放　>>>忘れずに!!<<<
delete [] packet_buf
```

### パケット受信サンプル(Deviceモード)
```cpp
// MIT License
// Copyright(c) 2025 NextAmusement
// See LICENSE file in the root directory.

// デバイスモードで初期化(ID:0x01 Ver:0x10);
ASerialPacket device(0x01, 0x10);

//メインループ(無限ループ)
void loop(void) {
    //受信したパケットデータを格納する構造体の宣言
    static ASerialDataStructd::ASerialData read_data_buf;

    int read_data = /*~~~環境ごとの受信バッファ読み取り処理~~~*/;
    int st = 0; //読み取りステータス格納変数宣言

    //読み取りが正常にできたらデータをReadPacketData()に渡す
    if(read_data > -1) {
        st = device.ReadPacketData((uint8_t)read_data, read_data_buf);
    }

    //stが1になるとパケットの読み取り完了(読み取り中は0、エラー時は-1)
    if(st == 1) {
        switch(read_data_buf.command) {
            case 0x00: //デバイスリセット(予約コマンド)
                //~~~デバイスのリセット~~~
                break;
            case 0x01: //デバイス情報の送信
                //~~~デバイス情報の送信~~~
                break;
            //ユーザー定義コマンド1(例:送信されたデータを元にLEDを操作する)
            case 0x20:
                if(read_data_buf.data[0] == 0) { //データが0ならLEDを消灯
                    //~~~LEDを消灯~~~
                }
                else {　//データが0以外ならLEDを点灯
                    //~~~LEDを点灯~~~
                }
                break;
            case 0x21: //ユーザー定義コマンド2
                //~~~ユーザーが定義した処理~~~
                break;
            case 0x22: //ユーザー定義コマンド3
                //~~~ユーザーが定義した処理~~~
                break;
        }
    }

    //~~~他のマイコン処理~~~
}

```

### データ送信パケット生成サンプル(Controllerモード) 

```cpp
// MIT License
// Copyright(c) 2025 NextAmusement
// See LICENSE file in the root directory.

// コントローラモードで初期化(ターゲットデバイスID:0x01);
ASerialPacket controller(0x01);

const uint8_t command = 0x20; //送信するコマンド
const size_t DATA_NUM = 3;  //データ数
uint8_t sampleData[DATA_NUM] = {10, 20, 30}; //送信するデータ

//パケット格納バッファサイズの取得
int buf_size = controller.GetNeedPacketBufSize(sampleData, DATA_NUM);

//パケット格納バッファの確保
uint8_t *packet_buf = new uint8_t[buf_size];

// packet_buf にパケットが格納される
controller.MakePacketData(sampleData, DATA_NUM, command, packet_buf);

//~~~packet_bufを環境ごとの方法で送信する~~~

//確保したバッファの開放　>>>忘れずに!!<<<
delete [] packet_buf
```

### データ送信パケット(コマンドのみ)生成サンプル(Controllerモード) 

```cpp
// MIT License
// Copyright(c) 2025 NextAmusement
// See LICENSE file in the root directory.

// コントローラモードで初期化(ターゲットデバイスID:0x01);
ASerialPacket controller(0x01);

const uint8_t command = 0x20; //送信するコマンド

//パケット格納バッファサイズの取得
const int buf_size = 6; //コマンドのみの場合はバッファサイズは'6'固定

//パケット格納配列生成
uint8_t packet_buf[buf_size] = {0};

// packet_buf にパケットが格納される
controller.MakePacketData(command, packet_buf);

//~~~packet_bufを環境ごとの方法で送信する~~~
```

### パケット受信サンプル(Controllerモード):ループ毎に読み取りタイプ(並列処理に強い)
```cpp
// MIT License
// Copyright(c) 2025 NextAmusement
// See LICENSE file in the root directory.

// コントローラモードで初期化(ターゲットデバイスID:0x01);
ASerialPacket controller(0x01);

//~~~センサの数値取得コマンド送信~~

//受信したパケットデータを格納する構造体の宣言
ASerialDataStructd::ASerialData read_data_buf;

//===以下をパケット読み取り完了するまで毎ループごとに呼び出す===

int read_data = /*~~~環境ごとの受信バッファ読み取り処理~~~*/;
int st = 0; //読み取りステータス格納変数宣言

//読み取りが正常にできたらデータをReadPacketData()に渡す
if(read_data > -1) {
    st = controller.ReadPacketData((uint8_t)read_data, read_data_buf);
}

//stが1になるとパケットの読み取り完了(読み取り中は0、エラー時は-1)
if(st == 1) {
    //読み取りを終了
}

//===以上===

//read_data_buf.commandには0x00が必ず格納される(コマンドは未使用のため)
//read_data_buf.data[] に受信したデータが格納される
//read_data_buf.data_num にデータ数が格納される
```

### パケット受信サンプル(Controllerモード):呼び出し完結タイプ
```cpp
// MIT License
// Copyright(c) 2025 NextAmusement
// See LICENSE file in the root directory.

// コントローラモードで初期化(ターゲットデバイスID:0x01);
ASerialPacket controller(0x01);

//~~~センサの数値取得コマンド送信~~

//受信したパケットデータを格納する構造体の宣言
ASerialDataStructd::ASerialData read_data_buf;

clock_t start_time = /*~~~現在の時間取得(ms)~~~*/;  //読み取り開始時の時間取得(環境依存)

while(1) {
    int read_data = /*~~~環境ごとの受信バッファ読み取り処理~~~*/;
    int st = 0; //読み取りステータス格納変数宣言

    //読み取りが正常にできたらデータをReadPacketData()に渡す
    if(read_data > -1) {
        st = controller.ReadPacketData((uint8_t)read_data, read_data_buf);
    }   

    //stが1になるとパケットの読み取り完了(読み取り中は0、エラー時は-1)
    //またはタイムアウトが発生したら
    if(st == 1 || /*~~~現在の時間取得(ms)~~~*/ - start_time >= 50) {
        //読み取りを終了
        break;
    }
}

//read_data_buf.commandには0x00が必ず格納される(コマンドは未使用のため)
//read_data_buf.data[] に受信したデータが格納される
//read_data_buf.data_num にデータ数が格納される
```