# ROBOBO
ICT 멘토링 엑스포 2021 한이음 공모전 로보보 팀 -장려상

## 프로젝트 개요 (Project Overview)

ROBOBO는 ICT 멘토링 엑스포 2021 한이음 공모전 출품작으로, 인공지능(AI)을 활용하여 제어되는 로봇 팔 시스템입니다. 이 프로젝트는 사용자의 손 움직임을 로봇 팔이 모방하도록 하는 것을 주요 기능으로 하며, 다양한 통신 방식과 제어 로직을 탐구하고 구현한 결과를 보여줍니다. 로봇 글러브에 장착된 센서를 통해 사용자의 손가락 움직임을 감지하고, 이 데이터를 로봇 팔로 전송하여 각 관절의 서보 모터를 정밀하게 제어합니다. 또한, 웹캠을 이용한 손 제스처 인식, AI 비전 센서(HuskyLens)를 통한 객체 인식 기반의 자율 동작 기능도 포함하고 있습니다. 클라우드(AWS IoT, AWS RDS) 및 웹 애플리케이션(Spring Boot)과의 연동을 통해 데이터 로깅, 사용자 관리 및 잠재적인 원격 제어 기능의 기반을 마련하였습니다.

## 시스템 아키텍처 (System Architecture)

ROBOBO 프로젝트는 다음과 같은 주요 구성 요소들로 이루어져 있으며, 각 요소들은 유기적으로 연동되어 로봇 팔 제어 시스템을 구성합니다.

*   **로봇 팔 (Robot Arm):** Arduino 기반의 로봇 팔로, 여러 개의 서보 모터로 구성되어 사람의 팔과 손의 움직임을 모방합니다.
*   **로봇 글러브 (Robot Glove):** Arduino 기반의 컨트롤러로, 플렉스 센서(flex sensor)가 장착되어 사용자의 손가락 움직임을 감지합니다.
*   **제어 시스템 (Control System):**
    *   **마이크로컨트롤러 (Microcontrollers):** 로봇 팔과 글러브에 사용되는 Arduino 보드들 (Nano, Nano 33 IoT/BLE 등).
    *   **싱글 보드 컴퓨터 (Single Board Computer):** Raspberry Pi가 글러브로부터 BLE 데이터를 수신하여 AWS IoT로 전송하거나, MQTT 브로커를 호스팅하는 등의 중간 제어 및 통신 허브 역할을 수행합니다.
    *   **PC/웹캠 (PC/Webcam):** MediaPipe 라이브러리를 이용한 손 제스처 인식을 수행하여 MQTT를 통해 특정 명령을 글러브로 전송합니다.
*   **통신 프로토콜 (Communication Protocols):**
    *   **BLE (Bluetooth Low Energy):** 글러브와 로봇 팔 간의 직접 통신 또는 글러브와 Raspberry Pi 간의 통신에 사용됩니다.
    *   **WiFi (Wireless Fidelity):**
        *   **AP (Access Point) 모드:** 글러브가 AP가 되어 로봇 팔이 직접 접속하여 데이터를 수신하는 방식입니다.
        *   **STA (Station) 모드:** 글러브나 로봇 팔이 기존 WiFi 네트워크에 접속하여 MQTT 브로커 또는 AWS IoT와 통신합니다.
    *   **MQTT (Message Queuing Telemetry Transport):** 로컬 네트워크 내에서의 경량 메시지 교환에 사용됩니다 (예: 손 제스처 인식 결과 전송). Mosquitto 브로커가 주로 Raspberry Pi에서 운영됩니다.
    *   **TCP/IP:** WiFi AP 모드 등에서 클라이언트-서버 통신에 사용됩니다.
    *   **AWS IoT Core:** Raspberry Pi에서 수집된 센서 데이터를 클라우드로 전송하고, 잠재적으로 클라우드에서 장치로 명령을 수신하는 데 사용됩니다.
*   **백엔드 서비스 및 데이터베이스 (Backend Services & Database):**
    *   **`rbb_springboot` (메인 웹 애플리케이션):** Spring Boot와 Java로 개발되었으며, 사용자 인증(Google/Facebook OAuth2), AWS RDS (MySQL) 데이터베이스 연동을 통한 데이터 관리 (사용자 정보, 로봇 설정, 로그 등) 및 웹 인터페이스를 제공합니다.
    *   **`Arduino/MQTT/springboot-mosquitto-master/` (보조 MQTT 웹 애플리케이션):** Spring Boot 기반의 별도 애플리케이션으로, Mosquitto MQTT 브로커와 연동하여 메시지 로깅, 관리 또는 웹을 통한 MQTT 테스트 등의 기능을 수행할 것으로 추정됩니다. (세부 기능은 코드 분석 제약으로 인해 명확히 파악되지 않음)
    *   **AWS RDS (MySQL):** `rbb_springboot` 애플리케이션의 주 데이터 저장소로 사용됩니다.

**데이터 흐름 및 제어 방식 요약:**

1.  **직접 제어:**
    *   **BLE 방식:** 글러브의 플렉스 센서 값 → BLE → 로봇 팔 서보 모터 직접 구동.
    *   **WiFi AP 방식:** 글러브의 플렉스 센서 값 → WiFi (글러브 AP, 로봇 팔 Client) → 로봇 팔 서보 모터 직접 구동.
2.  **간접/AI 제어:**
    *   **손 제스처 인식:** PC 웹캠 영상 → MediaPipe 손 제스처 인식 → MQTT 메시지 발행 → 로봇 글러브 수신 → 특정 동작 수행 (예: WiFi AP 모드로 전환).
    *   **HuskyLens AI 비전:** HuskyLens 카메라가 객체/태그 인식 → Arduino (로봇 팔) → 사전 정의된 팔 동작 수행.
3.  **클라우드 연동:**
    *   **센서 데이터 로깅:** 글러브 센서 값 → BLE → Raspberry Pi → AWS IoT Core로 발행.
    *   **(잠재적) 클라우드 기반 제어:** 외부 명령/웹 인터페이스 → AWS IoT Core → Raspberry Pi 수신 → 로봇 제어 (현재 `main_server.py`에서는 수신 메시지 출력만 구현됨).

*(참고: 시스템의 일부 세부 동작 및 Spring Boot 애플리케이션의 정확한 컨트롤러 로직은 현재 도구를 통한 코드 분석의 한계로 인해 추론된 내용을 포함하고 있습니다.)*

## 하드웨어 구성 요소 (Hardware Components)

ROBOBO 프로젝트의 로봇 팔과 글러브는 다음과 같은 주요 하드웨어 요소들로 구성됩니다.

### 1. 로봇 팔 (Robot Arm)

*   **마이크로컨트롤러 (Microcontroller):**
    *   Arduino Nano 또는 이와 유사한 호환 보드가 주로 사용됩니다. (`robot_arm_final.ino`, `Server/Final/robot.ino`)
    *   HuskyLens AI 카메라 연동 시에도 Arduino 보드가 메인 컨트롤러 역할을 합니다. (`husky_arm_test.ino`)
*   **액추에이터 (Actuators):**
    *   **서보 모터 (Servo Motors):** 로봇의 각 관절(손가락, 손목 등)을 움직이기 위해 다수의 서보 모터가 사용됩니다. 일반적으로 5개 이상의 서보 모터가 손가락 및 손목 제어에 사용됩니다.
    *   **Adafruit PCA9685 PWM Servo Driver:** 다수의 서보 모터를 정밀하게 제어하기 위해 I2C 인터페이스 기반의 PWM 드라이버 모듈이 사용됩니다. 이를 통해 제한된 Arduino 핀으로도 여러 서보를 제어할 수 있습니다.
*   **AI 비전 센서 (AI Vision Sensor):**
    *   **HuskyLens AI Camera:** 객체 인식, 얼굴 인식, 라인 트래킹, 색상 감지, 태그 감지 등의 기능을 제공하는 사용하기 쉬운 AI 비전 센서입니다. `husky_arm_test.ino`에서 특정 ID로 학습된 객체(예: 가위, 바위, 보)를 인식하여 로봇 팔이 해당 제스처를 취하도록 하는 데 사용됩니다.
*   **통신 모듈 (Communication Modules):**
    *   **WiFi Module:** Arduino Nano 33 IoT와 같은 보드에 내장된 WiFi 모듈(주로 ESP32 기반의 WiFiNINA 라이브러리 사용) 또는 별도의 WiFi 모듈을 사용하여 네트워크에 연결합니다.
    *   **BLE Module:** Arduino Nano 33 BLE와 같은 보드에 내장된 BLE 모듈을 사용하여 다른 BLE 장치와 통신합니다.

### 2. 로봇 글러브 (Robot Glove)

*   **마이크로컨트롤러 (Microcontroller):**
    *   Arduino Nano 33 IoT 또는 Nano 33 BLE와 같이 WiFi 및 BLE 기능을 모두 지원하거나, 특정 통신 방식에 맞는 Arduino 보드가 사용됩니다. (`robot_glove_final.ino`, `Arduino/BLE/hand_forearm_BLE/robot_glove_code_BLE.ino`)
*   **센서 (Sensors):**
    *   **플렉스 센서 (Flex Sensors):** 사용자의 손가락 각 마디 또는 주요 관절에 부착되어 구부림 정도를 감지합니다. 일반적으로 5개의 플렉스 센서가 각 손가락의 움직임을 측정하는 데 사용됩니다. (A1-A5 아날로그 핀에 연결)
*   **통신 모듈 (Communication Modules):**
    *   **WiFi Module:** 내장 WiFi 모듈을 사용하여 AP 모드로 동작하거나 기존 WiFi 네트워크에 연결하여 MQTT 또는 TCP/IP 통신을 수행합니다.
    *   **BLE Module:** 내장 BLE 모듈을 사용하여 로봇 팔 또는 Raspberry Pi와 같은 BLE Central/Peripheral 장치와 통신합니다.
*   **(잠재적) 자이로/가속도 센서 (Gyro/Accelerometer):**
    *   `robot_arm_final.ino` 코드 내에 MPU9250 자이로 센서 관련 코드가 주석 처리된 것으로 보아, 손목이나 팔의 회전 및 기울기 감지를 위한 확장 가능성을 염두에 둔 것으로 보입니다. (실제 최종 구현에는 미포함되었을 수 있음)

## 소프트웨어 및 제어 로직 (Software and Control Logic)

ROBOBO 프로젝트의 동작은 다양한 소프트웨어 구성 요소와 제어 로직의 조합을 통해 구현됩니다.

### 1. 아두이노 스케치 (Arduino Sketches)

로봇 팔과 글러브의 핵심 동작은 Arduino C/C++로 작성된 스케치에 의해 제어됩니다. 주요 스케치와 기능은 다음과 같습니다.

*   **로봇 팔 제어 스케치:**
    *   `Arduino/robot_arm_final/robot_arm_final.ino`: WiFi를 통해 서버(글러브 AP 모드 또는 별도 서버)에 접속하여 플렉스 센서 값을 수신하고, PCA9685 PWM 드라이버를 이용해 5개의 서보 모터(각 손가락)를 제어합니다. MPU9250 자이로 센서 코드가 주석 처리되어 있어 팔꿈치/어깨 제어 확장 가능성을 시사합니다.
    *   `Server/Final/robot.ino`: `robot_arm_final.ino`와 매우 유사하며, WiFi(SSID "test")를 통해 `192.168.4.1` 서버에 접속하여 6개의 서보 모터(5개 손가락 + 1개 손목 추정)를 PCA9685로 제어합니다.
    *   `Arduino/BLE/hand_forearm_BLE/robot_arm_code_BLE.ino`: BLE Peripheral 장치로 동작하며 ("Arduino Nano 33 BLE (Peripheral)"), 5개의 BLE Characteristic을 통해 플렉스 센서 값을 수신하여 5개 서보 모터를 직접 구동합니다.
    *   `Arduino/WIFI/hand_forearm_WIFI/robot_arm_code_wifi.ino`: `robot_arm_final.ino`와 유사하나, Servo 라이브러리를 직접 사용하며 PCA9685 드라이버를 사용하지 않는 버전으로 보입니다. `192.168.4.1` 서버에 접속합니다.
    *   `Arduino/huskylens/husky_arm_test.ino`: HuskyLens AI 카메라와 시리얼 통신하여, 인식된 객체 ID (1: 보, 2: 바위, 3: 가위)에 따라 PCA9685로 제어되는 서보 모터들을 움직여 해당 손 모양을 만듭니다.

*   **로봇 글러브 제어 스케치:**
    *   `Arduino/robot_glove_final/robot_glove_final.ino`: WiFi(STA 모드) 및 MQTT (PubSubClient 라이브러리) 통신 기능을 구현합니다. MQTT 브로커(`192.168.0.8`)에 접속하고 "send" 토픽을 구독합니다. "1" 메시지 수신 시 WiFi AP 모드(SSID "test", IP `192.168.4.1`)로 전환하여 로봇 팔에 직접 데이터를 전송할 수 있도록 서버 역할을 합니다. 5개의 플렉스 센서(A1-A5) 값을 읽습니다.
    *   `Arduino/BLE/hand_forearm_BLE/robot_glove_code_BLE.ino`: BLE Central 장치로 동작하며 ("Nano 33 BLE (Central)"), "Arduino Nano 33 BLE (Peripheral)" 장치를 스캔하여 연결합니다. 5개의 플렉스 센서 값을 읽어 연결된 로봇 팔의 BLE Characteristic에 값을 씁니다.
    *   `Arduino/WIFI/hand_forearm_WIFI/robot_glove_code_wifi.ino`: WiFi를 통해 `192.168.4.1` 서버(로봇 팔)에 플렉스 센서 값을 전송하는 클라이언트 역할을 합니다.

*   **주요 사용 라이브러리:** `WiFiNINA`, `Servo`, `Adafruit_PWMServoDriver`, `ArduinoBLE`, `PubSubClient`, `HUSKYLENS`.

### 2. 손 제스처 인식 (`HandDetection/HandDetection.py`)

*   Python 스크립트로, OpenCV와 MediaPipe 라이브러리를 사용하여 웹캠으로부터 실시간 영상을 입력받아 손의 랜드마크를 감지합니다.
*   감지된 손가락의 펴짐/접힘 상태를 분석하여 "주먹(rock)", "가위(scissor)", "보(paper)" 제스처를 인식합니다.
    *   주먹 (0개 폄): MQTT 메시지 "1" 발행
    *   가위 (2개 폄): MQTT 메시지 "2" 발행
    *   보 (5개 폄): MQTT 메시지 "3" 발행
*   인식된 제스처에 해당하는 MQTT 메시지를 `mosquitto_pub` 명령을 통해 로컬 Mosquitto 브로커(기본 IP `192.168.0.15`, 토픽 "send")로 발행합니다.
*   *참고: `robot_glove_final.ino`에서 사용하는 MQTT 브로커 IP(`192.168.0.8`)와 `HandDetection.py`에서 사용하는 IP(`192.168.0.15`)가 다릅니다. 실제 연동을 위해서는 이 주소가 일치해야 합니다.*

### 3. Raspberry Pi 서버 (`Server/server/main_server.py`)

*   Raspberry Pi에서 실행되는 Python 스크립트로, 다양한 통신 인터페이스 간의 브릿지 및 클라우드 연동 기능을 수행합니다.
*   **BLE 데이터 수신:** `bluepy` 라이브러리를 사용하여 특정 MAC 주소(`D2:0E:20:DA:1B:31`)를 가진 BLE Peripheral 장치(로봇 글러브로 추정)로부터 센서 데이터를 수신합니다.
*   **AWS IoT Core 연동:** `AWSIoTPythonSDK`를 사용하여 AWS IoT Core에 연결합니다.
    *   수신된 BLE 센서 데이터를 JSON 형식으로 포맷하여 AWS IoT 토픽 "server/write"로 발행(publish)합니다.
    *   AWS IoT 토픽 "server/read"를 구독(subscribe)하며, 메시지 수신 시 해당 내용을 콘솔에 출력합니다. (현재는 수신 데이터에 대한 구체적인 로봇 제어 로직은 없음)
*   멀티프로세싱(`multiprocessing`)을 사용하여 BLE 데이터 수신/AWS 발행 작업과 AWS 구독 작업을 병렬로 처리합니다.

### 4. Mosquitto MQTT 브로커

*   `Arduino/MQTT/Readme.md`에 설명된 바와 같이, 로컬 네트워크 내에서 경량 메시지 교환을 위한 MQTT 브로커로 Mosquitto가 사용됩니다.
*   주로 Raspberry Pi에 설치되어 운영될 것으로 예상됩니다 (`sudo apt install mosquitto mosquitto-clients`).
*   `HandDetection.py`가 발행자(publisher) 역할을, `robot_glove_final.ino`가 구독자(subscriber) 역할을 수행하여 손 제스처에 따른 글러브 모드 변경 등의 기능을 구현하는 데 사용됩니다.

## 클라우드 및 웹 서비스 (Cloud and Web Services)

ROBOBO 프로젝트는 데이터 저장, 사용자 관리, 잠재적인 원격 제어 및 모니터링을 위해 클라우드 서비스와 웹 애플리케이션을 활용합니다.

### 1. AWS IoT Core

*   **역할:** Raspberry Pi (`main_server.py`)에서 수집된 로봇 글러브의 센서 데이터를 안전하게 수신하고 관리하기 위한 클라우드 메시징 서비스입니다.
*   **데이터 흐름:**
    *   글러브 (BLE) → Raspberry Pi (`main_server.py`) → AWS IoT Core (토픽 "server/write"로 발행).
*   **활용:** 현재 `main_server.py`는 "server/write" 토픽으로 데이터를 발행하고 "server/read" 토픽을 구독하여 수신 메시지를 출력하는 기능까지만 구현되어 있습니다. 향후 이 데이터를 웹 대시보드에 시각화하거나, "server/read" 토픽을 통해 클라우드에서 로봇으로 명령을 전송하는 기능으로 확장될 수 있습니다.
*   **인증:** `main_server.py` 내에 인증서 경로가 설정되어 있어 X.509 인증서를 사용한 보안 연결을 지원합니다.

### 2. `rbb_springboot` (메인 웹 애플리케이션)

*   **기술 스택:** Spring Boot (Java), Thymeleaf (템플릿 엔진), Spring Security, JPA/Hibernate (ORM).
*   **데이터베이스:** AWS RDS for MySQL (`springboot-db.chdtgbna3nig.us-east-1.rds.amazonaws.com:3306/ROBOBO_Management`)를 사용하여 데이터를 영구적으로 저장합니다.
*   **주요 기능 (추정):**
    *   **사용자 인증:** Google 및 Facebook OAuth2를 통한 소셜 로그인을 지원합니다 (`application.properties` 설정 기반). JWT(JSON Web Token) 언급으로 보아 API 인증에도 사용될 수 있습니다.
    *   **데이터 관리 및 대시보드:** AWS RDS에 저장된 사용자 정보, 로봇 관련 설정, AWS IoT를 통해 수집된 센서 데이터 로그 등을 관리하고 웹 페이지를 통해 시각화하는 대시보드 기능을 제공할 수 있습니다.
    *   **로봇 제어 인터페이스 (잠재적):** 웹 UI를 통해 로봇의 상태를 모니터링하거나, 특정 명령을 MQTT 또는 AWS IoT를 통해 로봇으로 전송하는 인터페이스를 제공할 가능성이 있습니다. (현재 코드 분석 도구의 한계로 인해 해당 컨트롤러의 상세 로직은 확인되지 않았습니다.)
*   **구동:** Gradle 빌드 시스템을 사용하며, 내장 Tomcat 또는 별도의 서블릿 컨테이너에서 실행됩니다.

### 3. `Arduino/MQTT/springboot-mosquitto-master/` (보조 Spring Boot MQTT 애플리케이션)

*   **기술 스택:** Spring Boot (Java). (세부 라이브러리 및 컨트롤러는 분석되지 않음)
*   **역할 (추정):**
    *   이 애플리케이션은 로컬 Mosquitto MQTT 브로커와 연동되는 클라이언트 역할을 할 것으로 보입니다.
    *   가능한 기능으로는 MQTT 메시지 로깅 (예: `HandDetection.py`에서 발행되는 제스처 명령을 DB에 기록), 특정 토픽 구독 및 해당 데이터를 메인 `rbb_springboot` 애플리케이션으로 전달(API 호출 또는 DB 직접 기록), 또는 간단한 웹 인터페이스를 통한 MQTT 메시지 발행/테스트 등이 있을 수 있습니다.
    *   `application.properties` 파일이 비어있어 구체적인 외부 서비스(DB 등) 연동 정보는 확인되지 않았습니다.
*   **(불확실성):** 이 애플리케이션의 정확한 기능과 메인 `rbb_springboot` 애플리케이션과의 관계는 코드 분석의 제약으로 인해 명확히 파악되지 않았습니다.

### 4. AWS RDS (MySQL)

*   **역할:** `rbb_springboot` 애플리케이션의 주 데이터 저장소로 사용되는 관리형 관계형 데이터베이스 서비스입니다.
*   **저장 데이터 (추정):** 사용자 계정 정보, 로봇 설정, 센서 데이터 로그, 시스템 이벤트 등 프로젝트 운영에 필요한 다양한 데이터를 저장합니다.

## 통신 프로토콜 요약 (Communication Protocols Summary)

ROBOBO 프로젝트는 다양한 구성 요소 간의 데이터 교환을 위해 다음과 같은 통신 프로토콜들을 활용합니다.

*   **BLE (Bluetooth Low Energy):**
    *   **용도:** 로봇 글러브와 로봇 팔 간의 저전력, 근거리 무선 통신. 글러브의 센서 데이터를 팔로 직접 전송하거나, 글러브의 센서 데이터를 Raspberry Pi로 전송하는 데 사용됩니다.
    *   **예시:** `Arduino/BLE/hand_forearm_BLE/` 경로의 `robot_glove_code_BLE.ino` (Central)와 `robot_arm_code_BLE.ino` (Peripheral) 간의 통신. `Server/server/main_server.py`가 BLE Peripheral(글러브)로부터 데이터를 수신.

*   **WiFi (Wireless Fidelity):**
    *   **용도:** 로컬 네트워크를 통한 데이터 통신 및 인터넷 연결.
    *   **AP (Access Point) 모드:** 로봇 글러브(`robot_glove_final.ino`)가 자체 WiFi AP(SSID "test", IP `192.168.4.1`)를 생성하고, 로봇 팔(`robot_arm_final.ino` 또는 `Server/Final/robot.ino`)이 클라이언트로 접속하여 글러브로부터 직접 데이터를 수신합니다.
    *   **STA (Station) 모드:** 로봇 글러브나 로봇 팔이 기존의 WiFi 공유기에 접속하여 MQTT 브로커, AWS IoT Core 등 다른 네트워크 서비스와 통신합니다.
    *   **라이브러리:** 주로 `WiFiNINA` (Arduino Nano 33 IoT 등에서 사용)

*   **MQTT (Message Queuing Telemetry Transport):**
    *   **용도:** 경량의 발행-구독(Publish-Subscribe) 메시징 프로토콜로, 주로 로컬 네트워크 내에서 실시간 이벤트나 명령어 전달에 사용됩니다.
    *   **예시:** `HandDetection/HandDetection.py`에서 인식된 손 제스처 명령을 MQTT 토픽("send")으로 발행하면, `robot_glove_final.ino`가 이를 구독하여 특정 동작(예: AP 모드 전환)을 수행합니다.
    *   **브로커:** Mosquitto가 주로 Raspberry Pi에서 운영됩니다.

*   **TCP/IP:**
    *   **용도:** 신뢰성 있는 양방향 연결을 제공하는 표준 인터넷 프로토콜. WiFi AP 모드에서 글러브(서버)와 로봇 팔(클라이언트) 간의 데이터 전송에 기반 프로토콜로 사용됩니다.
    *   **예시:** `robot_glove_final.ino` (AP 모드에서 서버로 동작)와 `robot_arm_final.ino` (클라이언트로 접속) 간의 통신.

*   **Serial Communication:**
    *   **용도:** Arduino와 연결된 센서(예: HuskyLens) 또는 PC 간의 유선 데이터 교환.
    *   **예시:** `Arduino/huskylens/husky_arm_test.ino`에서 HuskyLens AI 카메라와 Arduino 간의 통신. Arduino IDE의 시리얼 모니터를 통한 디버깅.

*   **HTTPS/WSS (HTTP Secure / Secure WebSockets):**
    *   **용도:** AWS IoT Core 및 `rbb_springboot` 웹 애플리케이션과의 안전한 클라우드 및 웹 통신을 위해 사용됩니다. 데이터 암호화를 통해 보안을 강화합니다.
    *   **예시:** `Server/server/main_server.py`가 AWS IoT Core에 연결하거나, 웹 브라우저가 `rbb_springboot` 애플리케이션과 통신할 때 적용됩니다.

## 설정 및 설치 (Setup and Installation)

ROBOBO 프로젝트의 각 구성 요소를 실행하고 개발하기 위한 기본적인 설정 및 설치 가이드입니다.

### 1. 아두이노 (Arduino)

*   **Arduino IDE:** 아두이노 스케치(.ino 파일)를 컴파일하고 보드에 업로드하기 위해 Arduino IDE가 필요합니다. [공식 웹사이트](https://www.arduino.cc/en/software)에서 다운로드할 수 있습니다.
*   **라이브러리 설치:**
    *   Arduino IDE의 라이브러리 매니저를 통해 다음 라이브러리들을 검색하여 설치해야 할 수 있습니다 (스케치에서 `include`하는 라이브러리 기준):
        *   `WiFiNINA` (Arduino Nano 33 IoT 등 WiFi 기능이 있는 보드용)
        *   `Servo` (서보 모터 제어)
        *   `Adafruit_PWMServoDriver` (Adafruit PCA9685 PWM 드라이버 보드용)
        *   `ArduinoBLE` (Arduino Nano 33 BLE 등 BLE 기능이 있는 보드용)
        *   `PubSubClient` (MQTT 통신용)
        *   `HUSKYLENS` (HuskyLens AI 카메라 연동용)
*   **보드 드라이버:** 사용하는 Arduino 보드(Nano, Nano 33 IoT, Nano 33 BLE 등)에 맞는 드라이버가 PC에 설치되어 있어야 합니다. Arduino IDE 설치 시 함께 설치되거나 별도로 설치해야 할 수 있습니다.

### 2. Raspberry Pi 및 Mosquitto MQTT 브로커

*   **Raspberry Pi OS:** Raspberry Pi에 운영체제(예: Raspberry Pi OS)가 설치되어 있어야 합니다.
*   **Mosquitto MQTT Broker 설치 (Raspberry Pi):**
    *   `Arduino/MQTT/Readme.md`에 설명된 대로 다음 명령을 사용하여 Mosquitto 브로커와 클라이언트를 설치합니다:
        ```bash
        sudo apt update
        sudo apt install mosquitto mosquitto-clients
        ```
    *   브로커 활성화 및 상태 확인:
        ```bash
        sudo systemctl enable mosquitto
        sudo systemctl status mosquitto
        ```
*   **Python 환경 (Raspberry Pi):** `Server/server/main_server.py` 스크립트 실행을 위해 Python 3가 필요하며, 다음 라이브러리들을 pip를 통해 설치해야 합니다:
    ```bash
    pip install AWSIoTPythonSDK bluepy
    ```

### 3. PC (손 제스처 인식용)

*   **Python 환경 (PC):** `HandDetection/HandDetection.py` 스크립트 실행을 위해 Python 3가 필요하며, 다음 라이브러리들을 pip를 통해 설치해야 합니다:
    ```bash
    pip install opencv-python mediapipe
    ```
*   **웹캠:** 손 제스처 인식을 위해 웹캠이 연결되어 있어야 합니다.
*   **Mosquitto 클라이언트 (선택 사항):** `HandDetection.py`는 `os.system("mosquitto_pub ...")`를 사용하여 MQTT 메시지를 발행합니다. 따라서 `mosquitto-clients`가 PC에 설치되어 있거나, Python MQTT 클라이언트 라이브러리(예: `paho-mqtt`)를 사용하도록 스크립트를 수정할 수 있습니다.

### 4. 웹 애플리케이션 (Spring Boot)

*   **Java Development Kit (JDK):** `rbb_springboot` 및 `Arduino/MQTT/springboot-mosquitto-master/` 애플리케이션을 빌드하고 실행하기 위해 JDK (Java 8 또는 11 이상 권장)가 필요합니다.
*   **Gradle:** 프로젝트 빌드 및 의존성 관리를 위해 Gradle이 사용됩니다. 프로젝트 내에 `gradlew` (Linux/macOS) 및 `gradlew.bat` (Windows) 래퍼 스크립트가 포함되어 있어 별도의 Gradle 설치 없이 사용할 수 있습니다.
    *   빌드 예시 (프로젝트 루트 디렉토리에서):
        ```bash
        ./gradlew build 
        ```
        (또는 `.\gradlew.bat build` for Windows)
*   **데이터베이스 설정:**
    *   `rbb_springboot` 애플리케이션은 AWS RDS (MySQL)에 연결하도록 `application.properties`에 설정되어 있습니다. 해당 RDS 인스턴스가 실행 중이고 접근 가능해야 합니다.
    *   로컬 테스트 시에는 로컬 MySQL 서버를 설치하고 `application.properties`의 `spring.datasource.url`, `username`, `password`를 수정하여 사용할 수 있습니다.

### 5. AWS 서비스

*   **AWS 계정:** AWS IoT Core 및 AWS RDS를 사용하기 위해서는 AWS 계정이 필요합니다.
*   **AWS IoT Core 설정:**
    *   `main_server.py`에서 사용할 사물(Thing), 정책(Policy), 인증서(Certificates)를 AWS IoT Core 콘솔에서 생성하고 설정해야 합니다.
    *   스크립트 내의 엔드포인트 주소 및 인증서 경로가 실제 환경과 일치해야 합니다.
*   **AWS RDS 설정:**
    *   `rbb_springboot` 애플리케이션에서 사용할 MySQL RDS 인스턴스를 생성하고, `application.properties`에 해당 엔드포인트, 데이터베이스 이름, 사용자 인증 정보를 정확히 기입해야 합니다.

## 기능 요약 및 주요 동작 시나리오 (Functionality Overview / How it Works)

ROBOBO 프로젝트는 다양한 기술 요소가 결합되어 여러 가지 방식으로 로봇 팔을 제어하고 데이터를 관리합니다. 주요 기능 및 동작 시나리오는 다음과 같습니다.

### 시나리오 1: BLE를 이용한 직접 실시간 제어

1.  **글러브 센싱:** 사용자가 착용한 로봇 글러브(`Arduino/BLE/hand_forearm_BLE/robot_glove_code_BLE.ino` - BLE Central)의 플렉스 센서들이 손가락 움직임을 감지합니다.
2.  **데이터 전송:** 글러브의 Arduino는 감지된 센서 값을 BLE를 통해 로봇 팔(`Arduino/BLE/hand_forearm_BLE/robot_arm_code_BLE.ino` - BLE Peripheral)로 실시간 전송합니다.
3.  **로봇 팔 구동:** 로봇 팔의 Arduino는 수신된 BLE 데이터를 기반으로 각 서보 모터를 제어하여 사용자의 손 움직임을 그대로 모방합니다.
    *   **특징:** 저지연, 근거리 직접 제어 방식입니다. 별도의 네트워크 인프라(공유기, 서버 등)가 필요 없습니다.

### 시나리오 2: WiFi AP 모드를 이용한 직접 실시간 제어

1.  **글러브 AP 모드:** 로봇 글러브(`Arduino/robot_glove_final/robot_glove_final.ino`)가 WiFi Access Point (AP) 모드로 동작하여 자체 네트워크(SSID: "test", IP: `192.168.4.1`)를 생성합니다.
2.  **글러브 센싱 및 서버 역할:** 글러브는 플렉스 센서 값을 읽고, AP에 연결하는 클라이언트(로봇 팔)에게 이 데이터를 제공하는 서버 역할을 합니다.
3.  **로봇 팔 클라이언트 접속:** 로봇 팔(`Arduino/robot_arm_final/robot_arm_final.ino` 또는 `Server/Final/robot.ino`)은 글러브가 생성한 WiFi AP에 클라이언트로 접속합니다.
4.  **데이터 수신 및 구동:** 로봇 팔은 글러브로부터 TCP/IP를 통해 플렉스 센서 데이터를 수신하고, 이를 바탕으로 서보 모터를 제어하여 손 움직임을 모방합니다.
    *   **특징:** BLE보다 통신 거리가 길 수 있으며, 글러브가 중심이 되어 제어합니다.

### 시나리오 3: 손 제스처 인식을 통한 모드 변경 (MQTT 활용)

1.  **제스처 인식 (PC):** PC에 연결된 웹캠을 통해 `HandDetection/HandDetection.py` 스크립트가 사용자의 손 제스처(예: 주먹)를 인식합니다.
2.  **MQTT 메시지 발행 (PC):** 인식된 제스처에 해당하는 명령(예: "1")을 MQTT 토픽("send")으로 Mosquitto 브로커 (예: `192.168.0.15`)에 발행합니다.
3.  **MQTT 메시지 구독 (글러브):** 로봇 글러브(`Arduino/robot_glove_final/robot_glove_final.ino`)는 동일한 Mosquitto 브로커(설정 IP: `192.168.0.8` - *IP 일치 필요*)의 "send" 토픽을 구독하고 있다가 메시지를 수신합니다.
4.  **글러브 모드 변경:** 수신된 MQTT 메시지(예: "1")에 따라 글러브는 특정 동작을 수행합니다. 예를 들어, WiFi AP 모드로 전환하여 시나리오 2와 같이 로봇 팔 제어를 시작할 수 있습니다.
    *   **특징:** AI(손 제스처 인식)를 통해 시스템의 동작 모드를 변경하는 간접 제어 방식입니다.

### 시나리오 4: HuskyLens AI 카메라를 이용한 자율 동작

1.  **객체/태그 인식 (HuskyLens):** 로봇 팔에 장착된 HuskyLens AI 카메라(`Arduino/huskylens/husky_arm_test.ino` 연동)가 사전에 학습된 객체, 색상, 또는 태그를 인식합니다.
2.  **데이터 전송 (HuskyLens → Arduino):** HuskyLens는 인식된 객체의 ID를 시리얼 통신을 통해 로봇 팔의 Arduino로 전송합니다.
3.  **사전 정의된 동작 수행 (로봇 팔):** 로봇 팔의 Arduino는 수신된 ID 값에 따라 미리 프로그래밍된 특정 손 모양(예: ID 1이면 '보자기', ID 2면 '주먹')을 서보 모터를 움직여 실행합니다.
    *   **특징:** 외부의 시각적 자극에 반응하여 로봇 팔이 자율적으로 특정 동작을 수행합니다.

### 시나리오 5: 센서 데이터 클라우드 로깅 (Raspberry Pi 및 AWS IoT)

1.  **글러브 센싱 (BLE):** BLE 기능이 있는 로봇 글러브가 플렉스 센서 값을 감지합니다. (이 시나리오에서는 글러브가 BLE Peripheral로 동작한다고 가정)
2.  **데이터 수집 (Raspberry Pi):** Raspberry Pi에서 실행되는 `Server/server/main_server.py` 스크립트가 BLE를 통해 글러브로부터 센서 데이터를 수신합니다.
3.  **클라우드 전송 (Raspberry Pi → AWS IoT):** Raspberry Pi는 수집된 데이터를 JSON 형식으로 AWS IoT Core의 "server/write" 토픽으로 발행합니다.
4.  **데이터 저장/분석 (클라우드):** AWS IoT Core로 전송된 데이터는 AWS 내의 다른 서비스(예: S3, DynamoDB, RDS 등 - 현재는 미구현)와 연동되어 저장, 분석, 시각화 등에 활용될 수 있습니다. `rbb_springboot` 애플리케이션이 이 데이터를 RDS에서 조회하여 사용자에게 보여줄 수 있습니다.
    *   **특징:** 로봇의 센서 데이터를 클라우드에 기록하여 장기적인 분석이나 원격 모니터링의 기반을 마련합니다.

### 시나리오 6: 웹 애플리케이션을 통한 관리 및 모니터링 (잠재적)

1.  **사용자 접속:** 사용자가 웹 브라우저를 통해 `rbb_springboot` 웹 애플리케이션에 접속하고, Google/Facebook 계정으로 로그인합니다.
2.  **데이터 조회/관리:** 사용자는 웹 대시보드를 통해 AWS RDS에 저장된 이전 센서 데이터 로그, 로봇 설정 등을 조회하거나 관리할 수 있습니다.
3.  **(잠재적) 원격 명령:** 웹 인터페이스를 통해 특정 명령을 내리면, 이 명령이 백엔드 시스템(Spring Boot)을 거쳐 AWS IoT Core 또는 직접 MQTT를 통해 Raspberry Pi나 로봇 디바이스로 전달되어 특정 동작을 수행하게 할 수 있습니다. (이 부분은 현재 코드상 명시적으로 구현된 내용은 부족하며, 확장 가능성으로 해석)
    *   **특징:** 사용자 친화적인 웹 인터페이스를 통해 프로젝트와 상호작용하고, 데이터를 관리하며, 잠재적으로 원격 제어를 수행할 수 있습니다.

## 디렉토리 구조 (Directory Structure Overview)

ROBOBO 프로젝트의 주요 파일들은 다음과 같은 디렉토리 구조로 정리되어 있습니다.

*   **`Arduino/`**:
    *   로봇 팔 및 로봇 글러브를 제어하는 다양한 Arduino 스케치(.ino 파일)와 관련 코드, 라이브러리, 문서 등이 포함되어 있습니다.
    *   각 서브 디렉토리(예: `BLE/`, `WIFI/`, `MQTT/`, `huskylens/`, `robot_arm_final/`, `robot_glove_final/`)는 특정 통신 방식이나 최종 버전의 코드를 구분하여 저장합니다.

*   **`HandDetection/`**:
    *   OpenCV와 MediaPipe를 사용한 Python 기반의 손 제스처 인식 관련 파일(`HandDetection.py`)이 위치합니다.
    *   관련 논문이나 테스트 이미지/영상도 포함될 수 있습니다 (`paper/`, `test/`).

*   **`Server/`**:
    *   주로 Raspberry Pi에서 실행될 것으로 예상되는 서버 사이드 Python 스크립트와 관련 파일들이 포함되어 있습니다.
    *   `Server/server/main_server.py`: BLE 데이터 수신 및 AWS IoT 연동을 담당하는 핵심 서버 스크립트입니다.
    *   `Server/Final/`: 최종 버전으로 보이는 Arduino 스케치 파일(로봇 팔용)도 포함되어 있습니다.
    *   `MQTT_TEST/`, `CLIENT_TEST/`: MQTT 및 기타 클라이언트 테스트용 스크립트들이 저장되어 있습니다.

*   **`rbb_springboot/`**:
    *   메인 Spring Boot 웹 애플리케이션의 소스 코드와 리소스 파일들이 위치합니다.
    *   `main/java/`: Java 소스 코드 (컨트롤러, 서비스, DTO, 엔티티 등).
    *   `main/resources/`: `application.properties` (애플리케이션 설정), `static/` (CSS, JS, 이미지 등 정적 파일), `templates/` (Thymeleaf HTML 템플릿) 등이 포함됩니다.
    *   `build.gradle`: Gradle 빌드 스크립트.

*   **`README.md`**:
    *   현재 읽고 있는 이 파일로, 프로젝트 전반에 대한 설명과 기술 명세를 제공합니다.

*(참고: `Arduino/MQTT/springboot-mosquitto-master/` 와 같이 일부 Arduino 서브 디렉토리 내에도 별도의 Spring Boot 프로젝트가 포함된 경우가 있습니다. 이는 특정 기능을 독립적으로 테스트하거나 모듈화하려는 시도로 보입니다.)*
