# 🎉 Arduino代码示例教程

欢迎大家参加 2025 年 秀种暑期实践课程！🌱

本教程由老师和助教团队 🧑‍🏫👩‍💻 精心编写，无论你是初学者还是有一定经验，都可以通过本教程快速完成开发板调试和代码工作 🛠️💡。

请大家按照本教程逐步操作。如果在学习或实践过程中遇到问题，欢迎随时在班级群或课上向老师、助教提问，我们会及时为大家解答与支持 🤝。

预祝大家实践学习顺利，收获满满！🚀✨

> 指导老师: 张小乐 & 助教团队: 蹇姚宇蝶  
> 2025年6月22日


# 甲烷与 GPS 数据采集与上传程序说明

## 1. 功能简介

本程序基于 Arduino D1 R32 开发板，实现以下功能：

- 读取气体传感器（甲烷 CH₄）数值
- 读取 GPS 模块的实时定位数据
- 通过 WiFi 将数据打包成 JSON 后上传到指定服务器接口
- 可实时在串口查看数据上传状态

适用于气体监测、环境巡检等物联网场景。

---

## 2. 硬件连接

| ESP32 引脚    | 连接设备    | 设备引脚     |
|---------------|-------------|--------------|
| RX2 (GPIO16)  | CH₄ 传感器  | TX（B）      |
| TX2 (GPIO17)  | CH₄ 传感器  | RX（A）      |
| GPS_RX (14)   | GPS 模块    | TX           |
| GPS_TX (27)   | GPS 模块    | RX（可悬空） |

- 用 USB 连接 ESP32 和电脑，进行烧录和串口调试。
- 推荐 CH₄ 传感器波特率 115200，GPS 波特率 9600。

---

## 3. 主要功能说明

代码结构简述：
- `setup()`：初始化串口、WiFi 和外部传感器
- `loop()`：周期性读取 CH₄ 和 GPS 数据，并上传至服务器
- 主要自定义函数说明：
    - `connectWiFi()`：连接无线网络
    - `readCH4value()`：读取甲烷浓度并做数据校验
    - `gpsRead()`/`parseGpsBuffer()`/`updateGpsCoords()`：读取和解析 GPS 数据
    - `uploadSensorData()`：打包并上传传感器数据到服务器

### 3.1 WiFi 连接

填写实际 WiFi 名称（`ssid`）和密码（`password`），程序将自动联网。

### 3.2 气体传感器数据读取

- 通过硬件串口2（GPIO16、GPIO17）采集甲烷浓度，采用帧头和 CRC 校验。
- 浓度单位为 ppm。

### 3.3 GPS 数据读取与解析

- 通过硬件串口1（GPIO14、GPIO27）接收 GPS 数据。
- 程序自动提取 `$GPRMC` 或 `$GNRMC` 帧，解析经纬度，转为十进制度。

### 3.4 数据上传

- 数据以 JSON 格式通过 HTTP POST 上传到服务器接口（`serverURL`）。
- `serverURL` 需根据实际后端调整。

### 3.5 串口输出

- 每秒采集并上传数据，同时串口输出气体浓度与定位信息，便于调试。

---

## 4. 代码分段详解

### 4.1 头文件与全局变量

```cpp
#include <HTTPClient.h>     // 用于 HTTP 请求
#include <WiFi.h>           // 用于 ESP32 WiFi 联网
#include <WiFiClient.h>     // HTTPClient 内部依赖
#include <HardwareSerial.h> // 多串口支持
#include <ArduinoJson.h>    // JSON 数据序列化


// 填写你的 WiFi 和服务器信息
const char* ssid = "你的WiFi名称";
const char* password = "你的WiFi密码";
const char* serverURL = "你的服务器地址";

String username = "你的用户名";
String device_name = "在网站中添加的设备名称";
String device_key = "在网站中设置的设备密钥";
````

**说明：**
导入 ESP32、WiFi、HTTP、JSON 等必需库，并填写 WiFi 及服务器参数。

---

### 4.2 引脚与串口定义

```cpp
// 定义使用PIN脚
#define RX2_PIN 16
#define TX2_PIN 17
#define GPS_RX_PIN 14
#define GPS_TX_PIN 27
#define DEBUG_BAUD 115200

HardwareSerial GasSerial(2);  // 2号串口（气体传感器）
HardwareSerial GpsSerial(1);  // 1号串口（GPS）
const int GAS_BAUDRATE = 115200; // 气体传感器的波特率
const int GPS_BAUDRATE = 9600; // GPS设备的波特率
```

**说明：**
指定每个模块使用的 ESP32 引脚和串口号，便于代码和实际连线对应。

---

### 4.3 GPS 数据解析相关变量与结构体

```cpp
// GPS解析定义
String gps_lat = "";
String gps_lon = "";
float lat_deg = 0.0;
float lon_deg = 0.0;

struct
{
    char GPS_Buffer[80];
    bool isGetData;
    bool isParseData;
    char latitude[11];
    char longitude[12];
    bool isUsefull;
} Save_Data;

const unsigned int gpsRxBufferLength = 600;
char gpsRxBuffer[gpsRxBufferLength];
unsigned int ii = 0;
```

**说明：**
用于缓冲和解析 GPS 串口数据。

---

### 4.4 WiFi 连接函数

```cpp
// 尝试连接到指定 WiFi，无线连接成功前会一直等待
void connectWiFi() {
    // 启动 WiFi 连接，使用你填写的 WiFi 名称（ssid）和密码（password）
    WiFi.begin(ssid, password);

    Serial.print("Connecting to WiFi"); // 在串口监视器上输出连接状态提示

    // 检查 WiFi 连接状态。如果未连接上，会每隔 500ms 等待并输出一个点（.），表示正在重试
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);              // 等待 0.5 秒
        Serial.print(".");       // 显示连接进度
    }

    // 只有当 WiFi 连接成功后，才会跳出循环，继续往下执行
    Serial.println("\nConnected to WiFi"); // 连接成功后提示
}
```

**说明：**
自动连接 WiFi，并输出连接状态。

---

### 4.5 甲烷气体传感器数据读取

```cpp
// 从气体传感器串口读取数据帧，解析并返回甲烷浓度（单位：ppm）
// 返回值：
//   >0   - 甲烷浓度（ppm）
//   -1   - 帧尾校验失败（数据丢包）
//   -2   - CRC 校验失败（数据异常）
//   -3   - 没有收到完整数据
float readCH4value() {
    // 检查气体传感器串口缓冲区，如果有数据则进入读取流程
    while (GasSerial.available() > 0) {
        int b = GasSerial.read(); // 读取一个字节

        // 检查是否遇到数据帧头 0xFE
        if (b == 0xFE) {
            // 数据帧格式为 11 字节，帧头之后还应有 10 个字节，等待数据到齐
            while (GasSerial.available() < 10) {
                delay(2); // 等待剩余数据到达，避免读到半包
            }

            uint8_t buffer[11];
            buffer[0] = 0xFE; // 第一个字节存帧头
            GasSerial.readBytes(buffer + 1, 10); // 读取接下来的 10 字节

            // 帧尾检查，要求最后一个字节为 0x16，否则判为无效帧
            if (buffer[10] != 0x16) return -1;

            // 按数据协议分解各字段
            uint8_t byte3    = buffer[3];
            uint8_t byte4    = buffer[4];
            uint8_t gas_high = buffer[5]; // 浓度高字节
            uint8_t gas_low  = buffer[6]; // 浓度低字节
            uint8_t adc_high = buffer[7];
            uint8_t adc_low  = buffer[8];
            uint8_t crc      = buffer[9]; // CRC 校验字节

            // 根据协议计算 CRC 校验值（部分字节之和）
            uint8_t calc_crc = byte3 + byte4 + gas_high + gas_low + adc_high + adc_low;

            // 校验数据包合法性
            if (calc_crc == crc) {
                // 解析浓度数据：高低字节合并，除以 100，乘以 500 得到 ppm
                float concentration = (gas_high * 256 + gas_low) / 100.0 * 500;
                return concentration;
            } else {
                // CRC 校验不通过，返回 -2
                return -2;
            }
        }
    }
    // 如果串口中没有完整数据包，返回 -3
    return -3;
}
```

**说明：**
读取并校验甲烷数据帧，返回气体浓度（ppm），不同负值代表不同错误类型。

---

### 4.6 GPS 坐标转换与更新

```cpp
float convertNMEACoords(const char* nmea, bool isLat) {
    float raw = atof(nmea);
    int degrees = int(raw / 100);
    float minutes = raw - (degrees * 100);
    return degrees + minutes / 60.0;
}

void updateGpsCoords() {
    if (Save_Data.isParseData && Save_Data.isUsefull) {
        gps_lat = String(Save_Data.latitude); gps_lat.trim();
        gps_lon = String(Save_Data.longitude); gps_lon.trim();
        lat_deg = convertNMEACoords(Save_Data.latitude, true);
        lon_deg = convertNMEACoords(Save_Data.longitude, false);
        Save_Data.isParseData = false;
        Save_Data.isUsefull = false;
    }
}
```

**说明：**
NMEA 格式经纬度转换为十进制度。

---

### 4.7 GPS 数据读取与解析

```cpp
// 原有代码
void parseGpsBuffer() {
    // 解析 GPRMC/GNRMC 帧，提取经纬度和有效性
    // 详细代码略，见原代码段
}

void gpsRead() {
    // 读取串口缓冲，定位帧头并缓存数据帧
}
```

**说明：**
处理 GPS 模块的串口原始数据流。

---

### 4.8 数据上传

```cpp
void uploadSensorData(float ch4_value, float latitude, float longitude) {
    if (WiFi.status() != WL_CONNECTED) {
        Serial.println("WiFi not connected");
        return;
    }
    HTTPClient http;
    http.begin(serverURL);
    http.addHeader("Content-Type", "application/json");

    StaticJsonDocument<512> doc;
    doc["username"] = username;
    doc["device_name"] = device_name;
    doc["device_key"] = device_key;
    doc["timestamp"] = "";
    doc["lat"] = latitude;
    doc["lon"] = longitude;
    JsonObject data = doc.createNestedObject("data");
    data["CH4"] = ch4_value;
    
    String output;
    serializeJson(doc, output);

    int httpResponseCode = http.POST(output);
    Serial.print("HTTP Response: ");
    Serial.println(httpResponseCode);
    if (httpResponseCode > 0) {
        String response = http.getString();
        Serial.println(response);
    }
    http.end();
}
```

**说明：**
把传感器数据包装为 JSON，通过 HTTP POST 上传到服务器。

---

### 4.9 setup() 与 loop() 主流程

```cpp
void setup() {
    Serial.begin(115200);
    GasSerial.begin(115200, SERIAL_8N1, RX2_PIN, TX2_PIN);
    GpsSerial.begin(9600, SERIAL_8N1, GPS_RX_PIN, GPS_TX_PIN);
    connectWiFi();
}

void loop() {
    float ch4_lel = readCH4value();
    if (ch4_lel < 0) {
      Serial.println("Warning: load invid data:"); Serial.println(ch4_lel);
    }

    gpsRead();
    parseGpsBuffer();
    updateGpsCoords();

    Serial.print("CH4: "); Serial.println(ch4_lel, 2);
    Serial.print("lat: "); Serial.println(lat_deg, 6);
    Serial.print("lon: "); Serial.println(lon_deg, 6);
    uploadSensorData(ch4_lel, lat_deg, lon_deg);

    delay(1000);
}
```

**说明：**
主循环中每秒读取传感器和 GPS 数据、打印并上传。

---

## 5. 完整代码

> 建议：直接复制后粘贴到 Arduino IDE，新手需注意按需调整 WiFi 名称、密码与服务器地址。

```cpp
#include <HTTPClient.h>
#include <WiFi.h>
#include <WiFiClient.h>
#include <HardwareSerial.h>
#include <ArduinoJson.h>

// 1. 填写你的 WiFi 和服务器信息
const char* ssid = "你的WiFi名称";
const char* password = "你的WiFi密码";
const char* serverURL = "你的服务器地址";

String username = "你的用户名";
String device_name = "在网站中添加的设备名称";
String device_key = "在网站中设置的设备密钥";

#define RX2_PIN 16
#define TX2_PIN 17
#define GPS_RX_PIN 14
#define GPS_TX_PIN 27
#define DEBUG_BAUD 115200

HardwareSerial GasSerial(2);  // 2号串口
HardwareSerial GpsSerial(1);  // 1号串口
const int GAS_BAUDRATE = 115200;
const int GPS_BAUDRATE = 9600;

String gps_lat = "";
String gps_lon = "";
float lat_deg = 0.0;
float lon_deg = 0.0;

struct
{
    char GPS_Buffer[80];
    bool isGetData;
    bool isParseData;
    char latitude[11];
    char longitude[12];
    bool isUsefull;
} Save_Data;

const unsigned int gpsRxBufferLength = 600;
char gpsRxBuffer[gpsRxBufferLength];
unsigned int ii = 0;

void connectWiFi() {
    WiFi.begin(ssid, password);
    Serial.print("Connecting to WiFi");
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println("\nConnected to WiFi");
}

float readCH4value() {
    while (GasSerial.available() > 0) {
        // 找帧头
        int b = GasSerial.read();
        if (b == 0xFE) {
            // 检查后面够不够10字节
            while (GasSerial.available() < 10) {
                delay(2); // 等待剩余数据到达
            }
            uint8_t buffer[11];
            buffer[0] = 0xFE;
            GasSerial.readBytes(buffer + 1, 10);

            if (buffer[10] != 0x16) return -1;
            uint8_t byte3 = buffer[3];
            uint8_t byte4 = buffer[4];
            uint8_t gas_high = buffer[5];
            uint8_t gas_low  = buffer[6];
            uint8_t adc_high = buffer[7];
            uint8_t adc_low  = buffer[8];
            uint8_t crc      = buffer[9];

            uint8_t calc_crc = byte3 + byte4 + gas_high + gas_low + adc_high + adc_low;
            if (calc_crc == crc) {
                float concentration = (gas_high * 256 + gas_low) / 100.0 * 500; // ppm
                return concentration;
            } else {
                return -2;
            }
        }
    }
    return -3;
}

float convertNMEACoords(const char* nmea, bool isLat) {
    float raw = atof(nmea);
    int degrees = isLat ? int(raw / 100) : int(raw / 100);
    float minutes = raw - (degrees * 100);
    return degrees + minutes / 60.0;
}


void updateGpsCoords() {
    if (Save_Data.isParseData && Save_Data.isUsefull) {
        gps_lat = String(Save_Data.latitude); gps_lat.trim();
        gps_lon = String(Save_Data.longitude); gps_lon.trim();
        lat_deg = convertNMEACoords(Save_Data.latitude, true);
        lon_deg = convertNMEACoords(Save_Data.longitude, false);
        Save_Data.isParseData = false;
        Save_Data.isUsefull = false;
    }
}

void parseGpsBuffer() {
    char *subString;
    char *subStringNext;
    if (Save_Data.isGetData) {
        Save_Data.isGetData = false;
        char usefullBuffer[2] = {0}; // 有效性
        for (int i = 0; i <= 6; i++) {
            if (i == 0) {
                if ((subString = strstr(Save_Data.GPS_Buffer, ",")) == NULL)
                    while(1); // error
            } else {
                subString++;
                if ((subStringNext = strstr(subString, ",")) != NULL) {
                    switch(i) {
                        case 2: memcpy(usefullBuffer, subString, min((int)(subStringNext - subString), 1)); break; // 有效性
                        case 3: {
                            int len = ((subStringNext - subString) < (sizeof(Save_Data.latitude) - 1)) ?
                                      (subStringNext - subString) : (sizeof(Save_Data.latitude) - 1);
                            memcpy(Save_Data.latitude, subString, len);
                            Save_Data.latitude[len] = '\0';
                            break;
                        }
                        case 5: {
                            int len = ((subStringNext - subString) < (sizeof(Save_Data.longitude) - 1)) ?
                                      (subStringNext - subString) : (sizeof(Save_Data.longitude) - 1);
                            memcpy(Save_Data.longitude, subString, len);
                            Save_Data.longitude[len] = '\0';
                            break;
                        }
                        default: break;
                    }
                    subString = subStringNext;
                    Save_Data.isParseData = true;
                    if(usefullBuffer[0] == 'A') Save_Data.isUsefull = true;
                    else if(usefullBuffer[0] == 'V') Save_Data.isUsefull = false;
                } else {
                    while(1); // error
                }
            }
        }
    }
}

void gpsRead() {
    while (GpsSerial.available()) {
        // char c = GpsSerial.read();
        // Serial.write(c); // 打印全部原始串口内容
        // gpsRxBuffer[ii++] = c;
        gpsRxBuffer[ii++] = GpsSerial.read();
        if (ii == gpsRxBufferLength) { memset(gpsRxBuffer, 0, gpsRxBufferLength); ii = 0; }
    }
    char* GPS_BufferHead;
    char* GPS_BufferTail;
    if ((GPS_BufferHead = strstr(gpsRxBuffer, "$GPRMC,")) != NULL || (GPS_BufferHead = strstr(gpsRxBuffer, "$GNRMC,")) != NULL ) {
        if (((GPS_BufferTail = strstr(GPS_BufferHead, "\r\n")) != NULL) && (GPS_BufferTail > GPS_BufferHead)) {
            memcpy(Save_Data.GPS_Buffer, GPS_BufferHead, GPS_BufferTail - GPS_BufferHead);
            Save_Data.isGetData = true;
            memset(gpsRxBuffer, 0, gpsRxBufferLength); ii = 0;
        }
    }
}

void uploadSensorData(float ch4_value, float latitude, float longitude) {
    if (WiFi.status() != WL_CONNECTED) {
        Serial.println("WiFi not connected");
        return;
    }

    HTTPClient http;
    http.begin(serverURL);
    http.addHeader("Content-Type", "application/json");

    StaticJsonDocument<512> doc;
    doc["username"] = username;
    doc["device_name"] = device_name;
    doc["device_key"] = device_key;
    doc["timestamp"] = "";
    doc["lat"] = latitude;
    doc["lon"] = longitude;

    JsonObject data = doc.createNestedObject("data");
    data["CH4"] = ch4_value;
    
    String output;
    serializeJson(doc, output);

    int httpResponseCode = http.POST(output);
    Serial.print("HTTP Response: ");
    Serial.println(httpResponseCode);
    if (httpResponseCode > 0) {
        String response = http.getString();
        Serial.println(response);
    }

    http.end();
}


void setup() {
    Serial.begin(115200);
    GasSerial.begin(115200, SERIAL_8N1, RX2_PIN, TX2_PIN);
    GpsSerial.begin(9600, SERIAL_8N1, GPS_RX_PIN, GPS_TX_PIN);
    connectWiFi();
}

void loop() {
    float ch4_lel = readCH4value();
    if (ch4_lel < 0) {
      Serial.println("Warning: load invid data:"); Serial.println(ch4_lel);
      }

    // GPS数据读取与解析
    gpsRead();
    parseGpsBuffer();
    updateGpsCoords();  // 更新全局经纬度字符串

    Serial.print("CH4: "); Serial.println(ch4_lel, 2);
    Serial.print("lat: "); Serial.println(lat_deg, 6);
    Serial.print("lon: "); Serial.println(lon_deg, 6);
    uploadSensorData(ch4_lel, lat_deg, lon_deg);

    delay(1000);
}
```

---

## 6. 常见问题与扩展建议

* 检查连线、供电、电平兼容性，确保模块正常工作。
* 支持多气体扩展、断线重连、数据补传等功能。
* 可与 Web 可视化仪表盘联动，便于实时监控。

---

- 如有任何问题，欢迎在班级群或课上向老师/助教提问！
- 欢迎有代码优化改进建议！
