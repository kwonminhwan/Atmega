# Atmega
# 🚗 자동차 기능 시뮬레이션 시스템 - 기말 프로젝트

**제작자**: 2019265006 권민환  
**프로젝트명**: 차량 내 주요 기능 제어 시스템 (ATmega128A 기반)

---

## 📌 프로젝트 개요

이 프로젝트는 ATmega128A 마이크로컨트롤러를 기반으로, 자동차의 시동, 기어 변속, 비상등, 와이퍼, 경고등 및 초기화 기능을 구현한 시뮬레이션 시스템입니다. 사용자는 스위치를 이용하여 기능을 직접 제어할 수 있으며, 각 동작은 전자 신호(PWM, ADC, 인터럽트 등)를 활용해 제어됩니다.

---

## 🎯 목표 기능

| 기능 번호 | 기능 설명 |
|-----------|------------|
| 1️⃣ | 시동을 키고 기어 변속 (전압을 높이고 버튼을 누르면 DC 모터 회전) |
| 2️⃣ | 비상등 켜기 (버튼을 누르면 삼색 LED 켜짐 / 누르고 있으면 꺼짐) |
| 3️⃣ | 와이퍼 동작 (버튼으로 오른쪽, 다른 버튼으로 왼쪽 회전) |
| 4️⃣ | 경고등 동작 (LED 켜짐 + FND에 `119` 출력 + 절전모드 진입) |
| 5️⃣ | 초기화 기능 (모든 LED 및 FND OFF) |

---

## 🧩 구성도

```text
[Switch] ---> [ATmega128A MCU] ---> [RGB LED / DC 모터 / FND / 서보모터]
                                   ↘ [인터럽트 / ADC / PWM 제어]
```

---

## 🧠 주요 코드 구성

### ▫️ run() 함수 - ADC 측정 및 전압 가공

```c
void run(void) {
  ADCSRA |= 0x40;
  while (!(ADCSRA & 0x10));
  ADCSRA |= 0x10;
  result = ADCL & 0x00ff;
  adc_high = ADCH & 0x0003;
  result |= adc_high << 8;
  d_data = (result / 1.023) * 3;
}
```

---

### ▫️ 외부 인터럽트 예시

```c
interrupt [EXT_INT4] void external_int4(void) {
  if (f_color == 0) PORTF = 0x00;
  else if (f_color == 1) PORTF = 0x01;
  else PORTF = 0x07;
  f_color++;
  if (f_color == 2) f_color = 0;
}
```

---

### ▫️ FND 출력 함수

```c
void FND(unsigned char i, unsigned char j, unsigned char k) {
  for (int time = 0; time < 3000; time++) {
    PORTG = 0x0d; PORTC = i; delay_ms(1);
    PORTG = 0x0b; PORTC = j; delay_ms(1);
    PORTG = 0x07; PORTC = k; delay_ms(1);
    PORTG = 0x0f;
  }
}
```

---

### ▫️ 메인 제어 로직 (main + 스위치 핸들링)

- **스위치 1**: 시동/기어 동작 → DC 모터 PWM 제어
- **스위치 2**: 비상등 → RGB LED 토글
- **스위치 3**: 와이퍼 → 서보모터 양방향 제어
- **스위치 4**: 기어 단계 전환 (전압 범위별 포트 출력)
- **스위치 5**: 전체 기능 초기화
- **스위치 6**: 경고등 + `FND(119)` + 절전모드 진입

> 자세한 핀 구성 및 레지스터 초기화는 코드에 포함된 `DDRx`, `PORTx`, `TCCRn` 등의 설정 참조

---

## 🧪 결과 요약

- 각 기능별 스위치 연동 성공
- PWM, ADC, 인터럽트 등 주요 기능 구현 완료
- RGB LED 색상 전환, FND 숫자 출력 등 실동작 확인
- 절전 모드 및 인터럽트 복귀 동작 정상 작동

---

## 💭 느낀 점

> 배웠던 기능들을 내가 직접 다시 작성하고 실험해보면서 좀 더 잘 알게 되었고 유익한 시간이었습니다. 만들 때 많이 어려워 고비를 많이 겪었지만, 하나둘 해결해 나가면서 기능이 작동할 때 뿌듯하고 좋았습니다.

---

## 📂 파일 구성 (예시)

```
project/
├── main.c                 # 전체 기능 구현 C 코드
├── README.md              # 설명 문서
└── report.pptx            # 발표용 PPT 자료
```

---

## 📌 사용 부품 및 환경

- **MCU**: ATmega128A
- **IDE**: CodeVisionAVR 또는 AVR Studio
- **입출력 장치**: 스위치 6개, RGB LED, FND, DC 모터, 서보모터
- **전원**: 외부 5V 공급
- **기능 제어**: PWM, ADC, 외부 인터럽트, 절전모드

