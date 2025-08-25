# 【ASerial 環境依存クラス実装ガイド】
## 目的：ASerial通信を行うために必要な機能群を抽象化し、ユーザーが任意の環境に合わせて設計できるようにする。
## 対象：Windows / Android / macOS / Linux / MCU など


## [想定役割]
このクラスは、ASerialPacketクラスをラップし、
OSやデバイス固有の通信I/Oや機器制御と接続するためのインターフェイスを提供します。
ASerialのコアクラスである`ASerialPacket`を継承して実装してください。

## [クラス命名規則]
### `ASerial_lib_<対応モード(Controller or Device)>_<環境名>`

- 例: `ASerial_lib_Controller_Win`, `ASerial_lib_Device_Android`
- モードが両方に対応する場合は、対応モードは省略可

## 実装方針(Deviceモード)
### 機能
- **通信インターフェイス接続** - 環境のインターフェイスをクラス内でハンドリングするためにインターフェイスのポインタを受け取る。
- **受信パケットの読み取り** - 受信バッファにデータが来ていることを確認して、そのデータを逐一パケット解析を行う。パケットの読み取りが完了した時点でフラグを立てる。
- **データ送信** - コントローラからのコマンドを受け取り、パケットを生成してインターフェイスを通じて送信する。
- **デバイス情報の送信** - コントローラからのデバイス情報リクエストに応じて、デバイスIDとバージョンを送信する。
- **リセット関数の設定** - コントローラからのリセット要求に応じて、デバイスをリセットするための関数ポインタを受け取る。


## 実装方針(Controllerモード)
### 機能
- **デバイス接続** - デバイスに対してデバイス情報リクエストを送信し、返ってきた情報を元にデバイスIDの照合とデバイスVerの照合を行い、デバイスとの接続を行う。
- **オートデバイス接続**　- 指定したデバイスIDのデバイスを自動探査し、接続を行う。
- **デバイス切断**　- デバイスとの接続を切断し、ポートを閉じる。
- **通信インターフェイス接続** - 環境のインターフェイスをクラス内でハンドリングするためにインターフェイスのポインタを受け取る。
- **受信パケットの読み取り** - 受信バッファにデータが来ていることを確認して、そのデータを逐一パケット解析を行う。パケットの読み取りが完了した時点でフラグを立てる。
- **データ送信**　- デバイスに対してコマンドを送信するためのパケットを生成し、インターフェイスを通じて送信する。
- **データ送信(コマンドのみ)** - コマンドのみを送信するためのパケットを生成し、インターフェイスを通じて送信する。

## [クラス定義例(Controllerモード・Deviceモード両対応 - Arduino(MCU系))]
-------------------------------
```cpp
class ASerial_Lib_MCU : public ASerialPacket {
  public:
    
    /// @brief ASerial_Lib_MCUコンストラクタ(deviceモード)
    /// @param device_id デバイスID
    /// @param device_ver デバイスVer
    ASerial_Lib_MCU(uint8_t device_id, uint8_t device_ver);
    
    /// @brief ASerial_Lib_MCUコンストラクタ(controllerモード)
    /// @param target_device_id ターゲットデバイスID
    ASerial_Lib_MCU(uint8_t target_device_id);

    /// @brief 通信のインターフェースのポインタを設定
    /// @param _interface インターフェイスのポインタ
    void SetInterfacePt(Stream *_interface);

    /// @brief リセット関数のポインタを設定
    /// @param func_pt リセット関数のポインタ
    void SetResetFuncPt(void (*func_pt)(void));

    /// @brief パケットデータを読み取り
    /// @param read_data_buf データを格納するバッファへのポインタ
    /// @return 読み取り中は0、エラーが発生した場合は-1、読み取り完了で1を返す。
    int ReadDataProcess(ASerialDataStruct::ASerialData *read_data_buf);

    /// @brief パケット送信
    /// @param command コマンド
    /// @param data データ配列
    /// @param data_num データ数
    /// @return 正常時は0、エラー時は-1を返す。
    int WriteData(uint8_t command, uint8_t *data, uint8_t data_num);

    /// @brief パケット送信
    /// @param data データ配列
    /// @param data_num データ数
    /// @return 正常時は0、エラー時は-1を返す。
    int WriteData(uint8_t *data, uint8_t data_num);

  private:
    Stream *m_interface = nullptr;          // ストリームクラスのポインタ格納
    void (*m_reset_func)(void) = nullptr;   // リセット関数のポインタ

    /// @brief デバイス情報を送信
    /// @return 正常時は0、エラー時は-1を返す。
    int WriteDeviceInfo(void);
};
```

## [クラス定義例(Controllerモードのみ - Windows)]
-------------------------------
```cpp
class ASerial_lib_Controller_Win : public ASerialPacket {
 public:
    ASerial_lib_Controller_Win(int target_device_id, int device_ver);

    ASerial_lib_Controller_Win(int target_device_id, int device_ver_min, int device_ver_max);

    int ConnectDevice(int COM_num);

    int AutoConnectDevice(void);

    int DisConnectDevice(void);

    void SetInterfacePt(WindowsSerial *interface_pt);

    int ReadDataProcess(ASerialDataStruct::ASerialData *read_data_buf);

    int WriteData(uint8_t command, uint8_t *data, uint8_t data_num);

    int WriteData(uint8_t command);

 private:
    WindowsSerial *m_inteface = nullptr;
    int m_device_ver_min = 0;
    int m_device_ver_max = 0;
};
```

-------------------------------
■ クラス設計上のヒント
-------------------------------

- モードは ASerialPacket::GetMode() で判定できる（デバイス or コントローラ）
- パケットは MakePacketData() で生成し、GetNeedPacketBufSize() でサイズを取得
- 送信後は delete[] によるバッファ解放を忘れずに
- 接続確立時には ASerialPacket::SetConnectStatus() を true に設定
- 切断時には ASerialPacket::SetConnectStatus() を false に設定

-------------------------------
■ 実装対象プラットフォーム別メモ
-------------------------------

[Windows]
- 通信: Win32API または C# SerialPort  
- I/O: ReadFile / WriteFile など

[Android]
- 通信: UsbSerial, BluetoothSocket  

[macOS / Linux]
- 通信: termios + read/write  

[MCU系（STM32 / ESP32）]
- 通信: HAL_UART / Serial  
