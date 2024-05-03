&nbsp;
&nbsp;
<p align="center">
  <img width="800" src="./figures/conecta_logo.png" />
</p> 

&nbsp;

# Conect2AI Freematics One+ Implementation


The objective of this repository is to host the Conect2AI adaptation of the Freematics One+ platform. Relative to the [original implementation](https://github.com/stanleyhuangyc/Freematics/blob/master/firmware_v5/telelogger/telelogger.ino), notable enhancements include the incorporation of an email transmission feature within the server routing mechanism, as well as the integration of clock synchronization functionality utilizing an [NTP (Network Time Protocol) server](https://github.com/arduino-libraries/NTPClient).


## :rocket: How to Execute

1 - Install [Visual Studio Code](https://code.visualstudio.com/)

2 - Install [PlatformIO](https://platformio.org/) (VSCode Extension)

3 - Clone this repository

```bash
git clone https://github.com/conect2ai/conect2freematics
```

4 - Open `freematics-autcloud/firmware_v5/telelogger` project folder in PlatformIO, as shown in the figure below.

<p align="center">
  <img width="800" src="./figures/platformio.png" />
</p> 


5 - Connect the Freematics One+ with the computer and power up it with the Freematics Emulator or in the vehicle.

6 - Compile, Upload and monitor serial (steps 1, 2 and 3, respectively in the figure below).

<p align="center">
  <img width="800" src="./figures/upload.png" />
</p> 

## :computer: Sensors configuration

You can modify the sensors and the times they are collected in [telelloger.ino file](./firmware_v5/telelogger/telelogger.ino)

```C
PID_POLLING_INFO obdData[]= {
  {PID_ENGINE_LOAD, 1},
  {PID_RPM, 1},
  {PID_MAF_FLOW, 1},
  {PID_INTAKE_TEMP, 1},
  {PID_TIMING_ADVANCE, 1},
  {PID_MAF_FLOW, 1},
};
```

You can find the list with all the sensors available to collect in the [OBD.h file](./libraries/FreematicsPlus/utility/OBD.h)

```C
// Mode 1 PIDs
#define PID_ENGINE_LOAD 0x04
#define PID_COOLANT_TEMP 0x05
#define PID_SHORT_TERM_FUEL_TRIM_1 0x06
#define PID_LONG_TERM_FUEL_TRIM_1 0x07
#define PID_SHORT_TERM_FUEL_TRIM_2 0x08
#define PID_LONG_TERM_FUEL_TRIM_2 0x09
#define PID_FUEL_PRESSURE 0x0A
#define PID_INTAKE_MAP 0x0B
#define PID_RPM 0x0C
#define PID_SPEED 0x0D
...
```

## :wrench: Network Configurations

All the information collected can be send to a server and can be stored locally in SD card. The hardware allows information to be sent both via Wi-Fi and 4G. You can configure things like Wi-Fi name and password, URL server, server port, protocol etc. in [config.h file](./firmware_v5/telelogger/config.h).

```C

/**************************************
* Configuration Definitions
**************************************/
// ...

// define your email
#define USER_EMAIL "email@email.com"

/**************************************
* Networking configurations
**************************************/
#ifndef ENABLE_WIFI
#define ENABLE_WIFI 0
// WiFi settings
#define WIFI_SSID "FREEMATICS"
#define WIFI_PASSWORD "PASSWORD"
#endif 

#ifndef SERVER_HOST
// cellular network settings
#define CELL_APN ""
// Freematics Hub server settings
#define SERVER_HOST "serverconect2ai.dca.ufrn.br"
#define SERVER_PROTOCOL PROTOCOL_HTTP
#endif

// SIM card setting
#define SIM_CARD_PIN ""

// HTTPS settings
#define SERVER_METHOD PROTOCOL_METHOD_POST
#define SERVER_PATH "/freematics"

#if !SERVER_PORT
#undef SERVER_PORT
#if SERVER_PROTOCOL == PROTOCOL_UDP
#define SERVER_PORT 8081
#elif SERVER_PROTOCOL == PROTOCOL_HTTP
#define SERVER_PORT 1880
#elif SERVER_PROTOCOL == PROTOCOL_HTTPS
#define SERVER_PORT 443
#endif
#endif
```

## :brain: Embedding Tiny Machine Learning Algorithms

Within the [telelogger.ino](./firmware_v5/telelogger/telelogger.ino) file, the processOBD function assumes the pivotal role of acquiring sensor data and buffering it for subsequent transmission to both the SD card and the designated server. Consequently, this enables various applications such as utilizing the captured data to populate a vector, which can serve as input for machine learning algorithms. Also, it is possbile to calculate soft-sensors inside this function.

```C
void processOBD(CBuffer* buffer)
{
  
  static int idx[2] = {0, 0};
  int tier = 1;
  for (byte i = 0; i < sizeof(obdData) / sizeof(obdData[0]); i++) {

    // calculate tier
    ...

    // process value
    byte pid = obdData[i].pid;
    if (!obd.isValidPID(pid)) continue;
    int value;
    if (obd.readPID(pid, value)) {
        obdData[i].ts = millis();
        obdData[i].value = value;

        buffer->add((uint16_t)pid | 0x100, ELEMENT_INT32, &value, sizeof(value));
    } else {
        timeoutsOBD++;
        printTimeoutStats();
        break;
    }
    if (tier > 1) break;
  }
  int kph = obdData[0].value;
  if (kph >= 2) lastMotionTime = millis();
}
```

## :page_facing_up: License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

# About Us

The [**Conect2AI**](http://conect2ai.dca.ufrn.br) research group is composed of undergraduate and graduate students from the Federal University of Rio Grande do Norte (UFRN) and aims to apply Artificial Intelligence (AI) and Machine Learning in emerging fields. Our expertise includes Embedded Intelligence and IoT, optimizing resource management and energy efficiency, contributing to sustainable cities. In energy transition and mobility, we apply AI to optimize energy use in connected vehicles and promote more sustainable mobility.
