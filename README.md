# 배달 로봇의 연산 분담과 실시간성 설계

| 장치 | 갱신 주기 | 1회 데이터 | 데이터율 |
| --- | --- | --- | --- |
| 2D 라이다 | 15Hz(66.67ms) | 360 점 × (거리 4 B + 세기 4 B) = 2880 B | 43.2 kB/s |
| RGB 카메라 | 60fps (16.67ms) | 1280 x 720 x3 B = 2,764,800 B | 165.9 MB/s ≈ 1,327 Mbps |
| IMU | 400Hz (2.5ms) | 가속도 3축 + 각속도 3축 × 4 B + 타임스탬프 8 B = 32 B | 32 × 400 = 12.8 kB/s |
| 바퀴 엔코더 | 2kHz (0.5ms) | 2륜 × 4 B = 8 B | 8 × 2,000 = 16 kB/s |
| LTE | - | - | 업링크 95Mbps , 다운로드 100Mbps |

## 두 장치를 구분한 속성

| 장치 | 구분 속성 | 값 |
|---|---|---|
| 라이다 | `ATTR{loop/backing_file}` | `*/lidar.img` |
| IMU | `ATTR{loop/backing_file}` | `*/imu.img` |

### 작성한 udev 규칙 2개 + 규칙 키 설명표

```
SUBSYSTEM=="block", KERNEL=="loop*", ATTR{loop/backing_file}=="*/lidar.img", SYMLINK+="robot_lidar", MODE="0660", GROUP="dialout"
SUBSYSTEM=="block", KERNEL=="loop*", ATTR{loop/backing_file}=="*/imu.img", SYMLINK+="robot_imu", MODE="0660", GROUP="dialout"
```

규칙 키 설명
| 키 | 설명 |
| --- | --- |
| `SUBSYSTEM` | 서브시스템. loop 장치는 블록 장치이므로 `block`이다. |
| `KERNEL` | 커널에 붙인 장치 이름이다. |
| `ATTR{loop/backing_file}` | 장치 속성. loop 장치의 실제 파일 경로이다. |
| `SYMLINK+=` | 추가 이름(심볼릭 링크)을 만든다. 원래 이름은 그대로 남는다. |
| `MODE` | 장치 노드의 권환 소유자 및 그룹 권한 설정한다. |
| `GROUP` | 장치 노드의 그룹 소유자 설정한다. |

`==`와 `=`와 `+=`의 차이
| 연산자 | 역할 |
|---|---|---|
| `==` | 비교. 이 규칙을 적용할지 말지 고르는 매칭. 하나라도 안 맞으면 그 줄 전체가 건너뛰어진다. |
| `=` | 대입. 값을 통째로 설정한다(앞의 값을 덮어쓴다). |
| `+=` | 추가. 리스트형 키에 값을 덧붙인다. 심볼릭 링크는 여러 개일 수 있으므로 `+=` 를 쓴다. |

한줄을 읽을 때 `==` 조건이 전부 맞으면 `=`/`+=` 동작을 수행하라로 읽는다.
