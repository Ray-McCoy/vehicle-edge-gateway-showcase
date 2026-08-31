# Project Roadmap

本文件区分已经完成的稳定能力与后续可选扩展。

---

## Completed

### Embedded Linux Gateway V1

- UART acquisition / framing / signal mapping
- multithreaded processing pipeline
- TCP publishing / heartbeat / reconnect
- graceful shutdown and deployment lifecycle

### STM32F103 + FreeRTOS Vehicle Node

- Task / Queue / ISR architecture
- STM32 telemetry → Linux Gateway
- Linux → STM32 command path
- actuator execution and resource validation

### Signal-to-Service and Dynamic Feature

- VehicleStateStore / VehicleSignalService
- BodyControlService / CommandDispatcher
- same-process UART full duplex
- external Dynamic Feature process over UDS
- runtime Feature replacement

### Hardware-Driven Demo

- EC11 speed input
- physical left-turn / gear buttons
- hardware-driven Feature A / Feature B
- physical input → Vehicle Service → Feature → actuator end-to-end demo
- TCP reconnect and graceful-shutdown demos

---

## Possible Future Extensions

- actuator state feedback
- multi-requester actuator coordination
- schema-driven vehicle service APIs
