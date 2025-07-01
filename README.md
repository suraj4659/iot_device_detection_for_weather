# iot_device_detection_for_weather

### 1)How many devices of each type are installed per room?

     SELECT 
    room, type, COUNT(*)
FROM
    d.iot_devices
GROUP BY 1 , 2
ORDER BY 1
