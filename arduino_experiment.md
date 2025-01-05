# Experiment Results

## 1. CO2 Levels in a Classroom (Chungnam National University)

**Description**:  
CO2 levels were measured for 1.5 hours during a lecture in a classroom. The graph below shows CO2 concentrations dropping sharply after ventilation was applied.

![image](https://github.com/user-attachments/assets/64046fb1-83ee-4a81-8db3-b16cba02aa19)

- In an empty room, CO2 levels remained stable at 700–900 ppm. However, levels increased significantly when people were present.
- CO2 levels rose proportionally with the number of people in the room, but ventilation (10–20 minutes) caused a sharp decrease.
- In high-density spaces, CO2 levels climbed back quickly after ventilation, indicating the need for longer ventilation times.

---

## 2. Measuring Carbonation Strength in Soft Drinks

**Description**:  
The carbonation strength of four different soft drinks was measured by comparing the change in CO2 levels. Each experiment lasted for 2 minutes, with the sensor positioned vertically at the can’s opening after it was opened.

- **Soft Drinks Used**:  
  Coca-Cola, Coca-Cola Zero, Pepsi, and Pepsi Zero.
  
  ![image](https://github.com/user-attachments/assets/c10b41c4-bce6-46fd-a8d3-21d484af2e96)

- **Experiment Video**: https://youtu.be/WmNrj-KVs4I

- **Results**:  
  The results were visualized as a bar chart for clarity:

  ![image](https://github.com/user-attachments/assets/7c9d3eb5-5e02-46ad-8f74-2355f6f205d0)

  Based on the results, the CO2 change was observed in the following order:  
  **Coca-Cola Zero > Pepsi > Coca-Cola > Pepsi Zero**.  

  This indicates that **Coca-Cola Zero** has the highest carbonation strength among the tested drinks.



---

## 3. Discord CO2 Alert System

**Description**:  
A Discord-based alert system was implemented to notify users when CO2 levels exceed certain thresholds. The following Python script handles the integration.

```python
# 디스코드 이산화탄소 경고 알림 코드입니다.

import serial
import requests

# 아두이노 시리얼 포트 설정
SERIAL_PORT = '/dev/ttyUSB0'  # 아두이노가 연결된 포트 확인 후 설정
BAUD_RATE = 9600  # 아두이노와 동일한 보드레이트 설정

# 임계값 설정
THRESHOLD_1 = 850
THRESHOLD_2 = 1000
RESET_THRESHOLD = 700  # 환기가 완료되었다고 알릴 농도 임계값

# 디스코드 웹훅 URL
DISCORD_WEBHOOK_URL = 'https://discord.com/api/webhooks/your_webhook_url_here'


class AlertManager:
    def __init__(self):
        self.alert_850_sent = False
        self.alert_1000_sent = False
        self.reset_alert_sent = False

    def reset_alerts(self):
        """알림 상태 초기화"""
        self.alert_850_sent = False
        self.alert_1000_sent = False
        self.reset_alert_sent = False


def send_discord_alert(message):
    """디스코드로 알림 메시지를 전송합니다."""
    data = {"content": message}
    response = requests.post(DISCORD_WEBHOOK_URL, json=data)
    if response.status_code == 204:
        print("[디스코드 알림] 메시지가 성공적으로 전송되었습니다.")
    else:
        print(f"[디스코드 알림] 메시지 전송 실패: {response.status_code}")


alert_manager = AlertManager()
ser = None  # 초기값 설정

try:
    # 시리얼 포트 열기
    ser = serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=1)
    print("아두이노 연결 완료")

    while True:
        if ser.in_waiting > 0:
            line = ser.readline().decode('utf-8').strip()  # 아두이노에서 데이터 읽기
            print(f"[아두이노 데이터] {line}")

            if line.startswith("CO2:"):
                co2_value = int(line.split(":")[1])
                print(f"[센서] 현재 CO2 농도: {co2_value} ppm")

                # 임계값 확인 및 알림 전송 (한 번만 전송)
                if co2_value > THRESHOLD_2 and not alert_manager.alert_1000_sent:
                    alert_message = f"[경고] CO2 농도가 {co2_value} ppm으로 임계값 1000 ppm을 초과했습니다!"
                    print(alert_message)
                    send_discord_alert(alert_message)
                    alert_manager.alert_1000_sent = True  # 알림 상태 업데이트

                elif co2_value > THRESHOLD_1 and not alert_manager.alert_850_sent and not alert_manager.alert_1000_sent:
                    alert_message = f"[주의] CO2 농도가 {co2_value} ppm으로 임계값 850 ppm을 초과했습니다!"
                    print(alert_message)
                    send_discord_alert(alert_message)
                    alert_manager.alert_850_sent = True  # 알림 상태 업데이트

                # CO2 농도가 RESET_THRESHOLD 이하로 떨어질 경우
                elif co2_value <= RESET_THRESHOLD:
                    if alert_manager.alert_850_sent or alert_manager.alert_1000_sent:
                        if not alert_manager.reset_alert_sent:
                            reset_message = f"[알림] CO2 농도가 {co2_value} ppm으로 환기가 완료되었습니다!"
                            print(reset_message)
                            send_discord_alert(reset_message)
                            alert_manager.reset_alert_sent = True  # 환기 완료 알림 상태 업데이트

                # 값이 임계값 아래로 떨어지면 일반 알림 상태 초기화
                if co2_value > RESET_THRESHOLD:
                    alert_manager.reset_alert_sent = False

except Exception as e:
    print(f"[오류] {e}")

finally:
    if ser and ser.is_open:  # ser 변수가 정의되었는지 확인
        ser.close()
        print("[센서] 시리얼 포트 닫힘")
```

#### Experiment Video: https://youtu.be/CX9EXaEwwg8


#### Results:
- Sent alerts when CO2 exceeded thresholds (850 ppm, 1000 ppm).
- Reset notifications when CO2 levels dropped below 700 ppm.
