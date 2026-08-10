# BgNetSt
Check internet status for MIT AI2 extension

# 背景網路狀態監測與防崩潰擴充功能 (Background Network Status Guard)

### 📌 簡介
**背景網路狀態監測服務** (`com.luckyh9h.bgnetst`) 是專為 MIT App Inventor 2 開發的非視覺擴充功能（Extension）。本元件專門為了解決「App 長駐背景時，因網路環境不確定而引發 MQTT 或 Web 請求閃退」的痛點而設計，完美相容 **Itoo Sky 4.5** 等背景多進程排程器。

本擴充功能透過啟動一個原生的 Android 前台服務（Foreground Service），在底層即時監測硬體連線，並利用 `NET_CAPABILITY_VALIDATED` 技術動態驗證真實的外網通暢度，將最新狀態透過跨進程安全檔案（SharedPreferences）共享，確保背景任務永不失聯、絕不閃退。

### ⚙️ 核心優勢
* **完美解耦畫面 (No-Screen)**：即使手機進入休眠（Doze Mode）或 `Screen1` 被系統徹底銷毀，背景網路監測與防護依然 100% 正常運作。
* **精準破除假連線**：能自動識破「Wi-Fi 訊號滿格，但實際上路由器沒有連外網路」或「電梯內訊號被鎖死」等傳統元件無法偵測的盲區。
* **跨進程相容 (Itoo 專用)**：徹底打破 Android 多進程間的記憶體孤島效應，確保 Itoo 複製出的獨立背景進程在呼叫時能讀到最精確、即時的數值。
* **背景全域防崩潰**：內建全域未捕捉異常攔截器（UncaughtExceptionHandler），能安靜吞掉背景執行緒中的極端異常，死活不讓系統跳出「應用程式已停止運作」的崩潰視窗。

---

### 📦 擴充功能資訊
* **專案封包名稱 (Package Name)**：`com.luckyh9h.bgnetst`
* **版本 (Version)**：`5.0`
* **類別 (Category)**：`Extension`
* **非視覺元件 (Non-visible)**：`True`
* **元件圖示資產 (Designer Icon)**：`aiwebres/indeter_q_box_24.png`

---

### 🔒 資訊清單自動宣告 (Manifest Declarations)
本元件在編譯時已內嵌自動注入註解。當您將元件匯入 MIT App Inventor 2 後，系統會自動在 App 的 `AndroidManifest.xml` 中補上以下必要宣告，無需手動修改：

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_SPECIAL_USE" />

<service 
    android:name="com.luckyh9h.bgnetst.BgNetSt$NetworkMonitorService"
    android:enabled="true"
    android:exported="false"
    android:foregroundServiceType="specialUse" />
```

---

### 🧱 積木功能說明 (Block Documentation)

#### 函數 (Functions)

| 積木名稱 | 功能描述 | 回傳類型 |
| :--- | :--- | :--- |
| **StartNetworkService** | 啟動背景網路監測前台服務。*強烈建議在 `Screen1.Initialize` 事件中第一時間呼叫。* | *無* |
| **StopNetworkService** | 徹底停止背景監測服務並釋放系統網路鉤子（Network Callback）。 | *無* |
| **IsInternetValid** | **【核心功能】** 同步查詢當前是否具備真實、可靠的外網通路。可安全地直接嵌入在任何高頻率觸發的 Timer（如 Itoo 背景時鐘或內建 Clock）最外層。 | `Boolean` (`true` 或 `false`) |

---

### 🚀 實務推薦積木邏輯 (Best Practice)

不論是在主畫面還是 Itoo 背景 Procedure 中，請務必將此判斷作為金鐘罩擋在最前面：

```text
當 Itoo_Timer 或 Clock_Timer 執行時：
  如果 ( 呼叫 BgNetSt1.IsInternetValid ) 則：
      // 【安全暢通區】此時 Android 底層保證外網絕對暢通
      如果 ( UrsPahoMqttClient1.IsConnected ) 則：
          呼叫 UrsPahoMqttClient1.Publish ( 你的資料 )
      否則：
          呼叫 UrsPahoMqttClient1.Connect (嘗試重連)
  否則：
      // 【危險斷網區】網路已斷開、或處於無外網的假連線狀態
      // 默默跳過這次 Timer 觸發，不執行任何 MQTT 指令！App 絕不崩潰！
```

---

### 📄 授權條款 (License)
本專案基於 **MIT License** 開源分享。您可以自由地修改、散佈及商業化使用，唯須保留原作者 `com.luckyh9h` 之技術專案歸屬。
