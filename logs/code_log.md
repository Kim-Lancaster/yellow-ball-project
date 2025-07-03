# Code Change Log

## 2025-06-17

| Time     | Language | Files Modified      | Change Description                           | Test Status | Notes/Result    |
| -------- | -------- | ------------------- | -------------------------------------------- | ----------- | --------------- |
| 09:15:00 | Python   | `bot_rmt_ctrl.py`   | Added LED-flash routine on voice command     | ✅ Passed    | 100% coverage   |
| 10:42:30 | C++      | `sensor_fusion.cpp` | Implemented complementary filter for ToF+IMU | ✅ Passed    | Stable at 50 Hz |
| 11:03:10 | Python   | `utils/network.py`  | Wrapped socket connect in retry logic        | ✅ Passed    | Reconnect OK    |

**See troubleshooting**: [2025-06-17 Code Errors](code-error-log.md#2025-06-17)
