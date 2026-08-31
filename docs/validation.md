# Validation

本文件展示 Vehicle Edge Gateway 当前最终版本的代表性验证范围与结果。完整原始实验日志保留在私有源码仓库中。

---

## 1. Hardware Input and Telemetry

最终 Demo 使用真实物理输入：

```text
EC11 encoder
→ speed 0..120
→ +/-5 per detent

Left-turn button
→ left_turn 0 / 1

Gear button
→ Forward +1 / Reverse -1
```

Timing：

```text
input polling          = 5 ms
button debounce        ≈ 30 ms
telemetry publication  = 100 ms
```

Representative results：

```text
encoder quadrature decoding        PASS
speed lower / upper clamp          PASS
left-turn press / hold behavior    PASS
gear 1 <-> -1                      PASS
fast encoder final-state accuracy  PASS
```

实机 telemetry 稳定映射为：

```text
speed     → Vehicle.Speed
gear      → Vehicle.Powertrain.Transmission.SelectedGear
left_turn → Vehicle.Body.Lights.DirectionIndicator.Left.IsSignaling
```

---

## 2. Dynamic Feature Behavior

### Feature A — Low-Speed Cornering Light

```text
left_turn == true
AND
speed <= 30
→ Corner Light ON
```

Representative physical test cases：

```text
speed=20 left_turn=1  → PB0 ON
speed=30 left_turn=1  → PB0 ON
speed=35 left_turn=1  → PB0 OFF
speed=30 left_turn=0  → PB0 OFF
```

### Feature B — High-Speed Headlight Demo

```text
speed >= 50
→ Headlight ON
```

Representative physical test cases：

```text
speed=45  → PB1 OFF
speed=50  → PB1 ON
speed=55  → PB1 ON
speed=45  → PB1 OFF
```

Feature B 用于 runtime replacement demo，不是量产自动大灯策略。

---

## 3. Signal-to-Service and Bidirectional Control

已验证：

```text
SignalRouter
→ VehicleStateStore latest-state update
→ VehicleSignalService typed query
```

以及：

```text
Feature / Service
↓
BodyControlService
↓
BodyCommand
↓
CommandDispatcher
↓
SerialPort WRITE
↓
STM32 ControlTask
```

正式 Gateway 同一进程内可持续 UART READ physical telemetry，并同步 UART WRITE actuator command。

---

## 4. Runtime Replacement, Failure and Lifecycle

Runtime replacement sequence：

```text
Feature A
↓ stop
Gateway remains running
↓
Feature B
```

验证期间：

```text
Gateway process unchanged
Gateway binary unchanged
STM32 firmware unchanged
```

Feature 被 `kill -9` 后 Gateway 仍存活，新 Feature 可以重新连接。

TCP Monitor 暂时退出后 Gateway 保持运行并自动重连；Monitor 恢复后 telemetry 继续发布。

Gateway 收到 SIGINT 后完成有序 shutdown 与运行期资源清理。

---

## 5. Final STM32 Resource Regression

### Stack High Water Mark

| Task | Configured | HWM | Estimated Peak Used |
| --- | ---: | ---: | ---: |
| Acquisition | 128 words | 79 | 49 words / 196 B |
| Communication | 256 words | 128 | 128 words / 512 B |
| Control | 128 words | 88 | 40 words / 160 B |
| UartRx | 128 words | 73 | 55 words / 220 B |

### Heap

```text
FreeRTOS heap_4 total       8192 B
Current free                4720 B
Minimum ever free           4096 B
```

### Runtime Counters

```text
UART TX OK             = 2091
UART TX Error          = 0
UART RX Drop           = 0
UART RX Re-arm Error   = 0
Valid Commands         = 3
Invalid Commands       = 0
```

### MCU Memory

```text
text    27580 B
data      112 B
bss     13808 B

FLASH   27696 / 65536 B = 42.26%
RAM     13920 / 20480 B = 67.97%
```

---

## 6. Build Verification

```text
STM32 Debug build                 PASS
ARM vehicle-edge-gateway build    PASS
```
