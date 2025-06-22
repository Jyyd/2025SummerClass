# 🎉 Arduino代码示例教程

欢迎大家参加 2025 年秀钟暑期实践课程！🌱

本教程由老师和助教团队 🧑‍🏫👩‍💻 精心编写，无论你是初学者还是有一定经验，都可以通过本教程快速完成开发板调试和代码工作 🛠️💡。

请大家按照本教程逐步操作。如果在学习或实践过程中遇到问题，欢迎随时在班级群或课上向老师、助教提问，我们会及时为大家解答与支持 🤝。

预祝大家实践学习顺利，收获满满！🚀✨

> 指导老师: 张小乐 & 助教团队: 蹇姚宇蝶  
> 2025年6月22日

# 甲烷/二氧化碳 + GPS 多气体采集 Pro 版（

## Pro 版本主要新增/变动

1. **支持多种气体同时采集与上传**
    - 除 CH₄ 传感器外，增加了 CO₂ 传感器的读取与上传
    - 数据上传 JSON 中包含 CH₄ 和 CO₂ 两项

2. **多种串口类型混用（软串口+硬串口）**
    - CH₄ 传感器：硬件串口2（GPIO16/17）
    - CO₂ 传感器：使用 SoftwareSerial 软串口（GPIO25/26）
    - GPS：硬件串口1（GPIO14/27）

3. **CO₂ 传感器数据读取（新增函数 `readCO2ppm()`）**

```cpp
float readCO2ppm() {
    // 检查 CO2 软串口缓冲区，有数据则尝试解析
    while (GasCO2Serial.available() > 0) {
        int firstByte = GasCO2Serial.read();
        if (firstByte == 0x2C) { // 帧头标志
            while (GasCO2Serial.available() < 5) delay(1); // 等剩余数据到齐
            uint8_t buffer[6];
            buffer[0] = 0x2C;
            GasCO2Serial.readBytes(buffer + 1, 5);

            // 帧数据校验
            uint8_t checksum = buffer[0] + buffer[1] + buffer[2] + buffer[3] + buffer[4];
            if (checksum == buffer[5]) {
                float co2_ppm = (buffer[1] * 256 + buffer[2]);
                return co2_ppm;
            } else {
                Serial.println("Checksum failed.");
                return -2;
            }
        } else {
            // 非帧头字节时滑动丢弃，并打印提示
            Serial.print("Not a frame head: ");
            Serial.println(firstByte, HEX);
        }
    }
    return -4; // 未接收到有效数据包
}
````

**说明：**

* 串口读取 CO₂ 数据帧（6字节），帧头为0x2C，末尾1字节为校验和。
* 数据格式通常参考具体传感器文档。
* 校验和错误、数据不足等都做了容错，便于调试。

---

4. **数据上传格式变化**

```cpp
void uploadSensorData(float ch4_value, float co2_ppm, float latitude, float longitude) {
    // ...省略部分代码...
    JsonObject data = doc.createNestedObject("data");
    data["CH4"] = ch4_value;
    data["CO2"] = co2_ppm; // 注意："CO2" 键建议无空格
    // ...其余代码同基础版...
}
```

**说明：**

* data 字段下 CH₄、CO₂ 数据都被打包进 JSON 上传
* 建议 `"CO2"`（无空格），防止后端解析问题

---

5. **主循环 loop() 逻辑更新**

```cpp
void loop() {
    float ch4_lel = readCH4value();
    float co2_ppm = readCO2ppm();
    if (ch4_lel < 0) {
      Serial.println("Warning CH4: load invid data:"); Serial.println(ch4_lel);
    }
    if (co2_ppm < 0) {
      Serial.println("Warning CO2: load invid data:"); Serial.println(co2_ppm);
    }
    gpsRead();
    parseGpsBuffer();
    updateGpsCoords();

    Serial.print("CH4: "); Serial.println(ch4_lel, 2);
    Serial.print("CO2: "); Serial.println(co2_ppm, 2);
    Serial.print("lat: "); Serial.println(lat_deg, 4);
    Serial.print("lon: "); Serial.println(lon_deg, 4);
    uploadSensorData(ch4_lel, co2_ppm, lat_deg, lon_deg);

    delay(1000);
}
```

**说明：**

* 每秒采集并上传 CH₄、CO₂、GPS 三种数据
* 若任何一种气体数据读取失败，串口给出警告（便于查找传感器连接与协议问题）

---

6. **串口初始化变化**

```cpp
void setup() {
    Serial.begin(115200);
    GasCH4Serial.begin(115200, SERIAL_8N1, CH4_RX_PIN, CH4_TX_PIN); // CH4 硬件串口
    GasCO2Serial.begin(9600); // CO2 软串口
    GpsSerial.begin(9600, SERIAL_8N1, GPS_RX_PIN, GPS_TX_PIN);
    connectWiFi();
}
```

**说明：**

* 增加 CO₂ 软串口初始化

---

7. **引脚分配变化（多模块并行接入）**

* CH₄（甲烷）传感器：GPIO16（RX2），GPIO17（TX2）
* CO₂（二氧化碳）传感器：GPIO25（RX），GPIO26（TX）（通过 SoftwareSerial）
* GPS：GPIO14（RX1），GPIO27（TX1）

---

## ⚠️ 注意事项

* **软件串口（SoftwareSerial）** 在 ESP32 上最多稳定支持 1 个，速度不宜过高，波特率建议 ≤ 9600。
* 硬件串口多用于高频/大数据量传感器，GPS和 CH₄ 采用硬串口更稳定。
* 上传时建议数据字段名称统一，避免空格或中英文混用。
* 设备唯一标识和权限管理通过 `username`、`device_name`、`device_key` 字段实现。

---
## 完整代码
```cpp
#include <HTTPClient.h>
#include <WiFi.h>
#include <WiFiClient.h>
#include <HardwareSerial.h>
#include <ArduinoJson.h>
#include <SoftwareSerial.h>


const char* ssid = "你的WiFi名称";
const char* password = "你的WiFi密码";
const char* serverURL = "你的服务器地址";

String username = "你的用户名";
String device_name = "在网站中添加的设备名称";
String device_key = "在网站中设置的设备密钥";

#define CH4_RX_PIN 16 // 传感器 TX -> ESP32 CH4_RX B
#define CH4_TX_PIN 17 // 传感器 RX -> ESP32 CH4_TX A
#define CO2_RX_PIN 25 // 传感器 TX -> ESP32 CO2_RX B
#define CO2_TX_PIN 26 // 传感器 RX -> ESP32 CO2_TX A
#define GPS_RX_PIN 14   // GPS   RX -> ESP32 GPS_TX
#define GPS_TX_PIN 27   // 不需要使用
#define DEBUG_BAUD 115200

HardwareSerial GpsSerial(1);  // 1号串口（GPS）
HardwareSerial GasCH4Serial(2);  // 2号串口（气体传感器）
SoftwareSerial GasCO2Serial(CO2_RX_PIN, CO2_TX_PIN);
const int GAS_CH4_BAUDRATE = 115200;  // 气体传感器实际波特率
const int GAS_CO2_BAUDRATE = 9600;  // 气体传感器实际波特率
const int GPS_BAUDRATE = 9600;    // GPS实际波特率

String gps_lat = "";
String gps_lon = "";
float lat_deg = 0.0;
float lon_deg = 0.0;

struct
{
    char GPS_Buffer[80];
    bool isGetData;		// 是否获取到GPS数据
    bool isParseData;	// 是否解析完成
    char latitude[11];	// 纬度
    char longitude[12];	// 经度
    bool isUsefull;		// 定位信息是否有效
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
    while (GasCH4Serial.available() > 0) {
        // 找帧头
        int b = GasCH4Serial.read();
        if (b == 0xFE) {
            // 检查后面够不够10字节
            while (GasCH4Serial.available() < 10) {
                delay(2); // 等待剩余数据到达
            }
            uint8_t buffer[11];
            buffer[0] = 0xFE;
            GasCH4Serial.readBytes(buffer + 1, 10);

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
                float concentration = (gas_high * 256 + gas_low) / 100.0 *  500;
                return concentration;
            } else {
                return -2;
            }
        }
    }
    return -3;
}


float readCO2ppm() {
    // 1. 寻找帧头
    while (GasCO2Serial.available() > 0) {
        int firstByte = GasCO2Serial.read();
        if (firstByte == 0x2C) {
            // 2. 等待剩余5字节
            while (GasCO2Serial.available() < 5) delay(1);
            uint8_t buffer[6];
            buffer[0] = 0x2C;
            GasCO2Serial.readBytes(buffer + 1, 5);

            uint8_t checksum = buffer[0] + buffer[1] + buffer[2] + buffer[3] + buffer[4];
            if (checksum == buffer[5]) {
                float co2_ppm = (buffer[1] * 256 + buffer[2]);
                return co2_ppm;
            } else {
                Serial.println("Checksum failed.");
                return -2;
            }
        } else {
            // 滑动丢弃字节，直到遇到0x2C
            Serial.print("Not a frame head: ");
            Serial.println(firstByte, HEX);
        }
    }
    return -4;
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

void uploadSensorData(float ch4_value, float co2_ppm, float latitude, float longitude) {
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
    data["CO2 "] = co2_ppm;

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
    GasCH4Serial.begin(115200, SERIAL_8N1, CH4_RX_PIN, CH4_TX_PIN);
    GasCO2Serial.begin(9600); // 软串口
    GpsSerial.begin(9600, SERIAL_8N1, GPS_RX_PIN, GPS_TX_PIN);
    connectWiFi();
}

void loop() {
    float ch4_lel = readCH4value();
    float co2_ppm = readCO2ppm();
    if (ch4_lel < 0) {
      Serial.println("Warning CH4: load invid data:"); Serial.println(ch4_lel);
      }
    if (co2_ppm < 0) {
      Serial.println("Warning CO2: load invid data:"); Serial.println(co2_ppm);
      }

    // GPS数据读取与解析
    gpsRead();
    parseGpsBuffer();
    updateGpsCoords();  // 更新全局经纬度字符串

    Serial.print("CH4: "); Serial.println(ch4_lel, 2);
    Serial.print("CO2: "); Serial.println(co2_ppm, 2);
    Serial.print("lat: "); Serial.println(lat_deg, 4);
    Serial.print("lon: "); Serial.println(lon_deg, 4);
    uploadSensorData(ch4_lel, co2_ppm, lat_deg, lon_deg);

    delay(1000);
}

```
- 如有任何问题，欢迎在班级群或课上向老师/助教提问！
- 欢迎有代码优化改进建议！
