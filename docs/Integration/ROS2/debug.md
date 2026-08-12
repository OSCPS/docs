# Counter Monitor - Quick Reference
Use this tool to **detect dropped IMU packets** and validate communication quality.

```bash
# Quick 60-second check
ros2 run oscp_imu_testing counter_monitor --ros-args -p duration:=60.0

# Expected output (good):
# /oscp/raw: 100.0 Hz (6000 msgs)
# Total missed packets: 0
```

---

## Common Commands

### Validate Before Deployment (5 minutes)
```bash
ros2 run oscp_imu_testing counter_monitor --ros-args -p duration:=300.0
```
**Expected:** Zero dropped packets, all topics at correct Hz.

### Monitor Continuously (until Ctrl+C)
```bash
ros2 run oscp_imu_testing counter_monitor
```

---

## Interpreting Output

### Console (Good)
```
/oscp/raw: 100.0 Hz (100 msgs)
/oscp/imu/data: 100.0 Hz (100 msgs)
... (no warnings)
Total missed packets: 0
```
✅ **Healthy communication**

### Console (Warning)
```
[WARN] Gap detected at 1234567 ms: last=42 expected=43 received=47 missed=4 (total=12)
```
⚠️ **4 packets dropped at timestamp 1234567 ms**

### CSV File
```
timestamp_ms,last_counter,expected_counter,received_counter,missed
1234567,42,43,47,4
1234800,99,100,105,5
```
Each row = one gap. Analyze patterns for root cause.

---

## Expected Hz Values

| Topic | LOW Mode | MEDIUM Mode | Status |
|-------|----------|-------------|--------|
| `/oscp/raw` | 100 Hz | 500 Hz | Always enabled |
| `/oscp/quat` | 100 Hz | 100 Hz | If enabled |
| `/oscp/euler` | 100 Hz | 100 Hz | If enabled |
| `/oscp/rot` | 100 Hz | 100 Hz | If enabled |
| `/oscp/gnss` | 1 Hz | 1 Hz | If enabled |
| `/oscp/imu/data` | 100 Hz | 100 Hz | If `publish_standard_ros=true` |
| `/oscp/imu/mag` | 100 Hz | 100 Hz | If `publish_standard_ros=true` |
| `/oscp/imu/temp` | 100 Hz | 100 Hz | If `publish_standard_ros=true` |

**0 Hz = Normal (topic not enabled)**  
**1 Hz = Normal (GNSS Frame)**
**100 Hz = Normal (LOW or MEDIUM AHRS)**  
**500 Hz = Normal (MEDIUM raw only)**

---

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| All topics at 0 Hz | IMU node not running | `ros2 launch oscp_imu_ros2 oscp_imu.launch.py` |
| Topic suddenly stops updating | Hardware disconnected | Check USB/CAN cable |

---

## Files Generated

| File | Contents |
|------|----------|
| `counter_gaps.csv` | One row per dropped packet |

---

## When to Use
- **Before field deployment** – Validate IMU works cleanly  
- **During sensor fusion tuning** – Correlate gaps with EKF divergence  
- **When debugging nav stack** – Rule out IMU communication issues  
- **After hardware changes** – Verify new cables/adapters  

