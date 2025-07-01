# iot_device_anomaly_detection_for_disaster(sql)



### 1)How many devices of each type are installed per room?
     SELECT 
    room, type, COUNT(*)
    FROM
    d.iot_devices
    GROUP BY 1 , 2
    ORDER BY 1
### 2) 📉 Which device type has the highest average anomaly rate (based on thresholds)?

    select id.type,
    count(case 
           when dl.metric_type = 'noise_level' and dl.value > 75 then 1
           when dl.metric_type = 'temperature' and dl.value > 35 then 1
           when dl.metric_type = 'brightness' and dl.value < 10 then 1

      end) as anonmaly
    from d.device_logs as dl 
    join d.iot_devices as id
    on dl.device_id = id.device_id 
    group by 1
    
### 3) Which IoT devices showed abnormal activity within 5 minutes before any disaster?

    SELECT 
     dl.device_id,
     id.type,
     id.room,
     dl.timestamp,
     dl.metric_type,
     dl.value,
     de.event_id
    FROM
           d.device_logs AS dl
               JOIN
             d.iot_devices AS id ON dl.device_id = id.device_id
              JOIN
             d.disaster_events AS de ON dl.timestamp BETWEEN DATE_SUB(de.timestamp,
              INTERVAL 3 MINUTE) AND de.timestamp
     WHERE
        (metric_type = 'temperature'
        AND value > 35)
        OR (metric_type = 'noise_level'
        AND value > 70)
        OR (metric_type = 'brightness'
        AND value < 10)

### 4)What is the average temperature in each room during the 10 minutes before disasters?

    SELECT 
       id.room, ROUND(AVG(dl.value), 2)
    FROM
        d.device_logs AS dl
            JOIN
        d.iot_devices AS id ON dl.device_id = id.device_id
            JOIN
        d.disaster_events AS de ON dl.timestamp BETWEEN DATE_SUB(de.timestamp,
           INTERVAL 10 MINUTE) AND de.timestamp
      WHERE
     metric_type = 'temperature'
      GROUP BY 1

### 5) Which disaster had the highest number of abnormal readings across all devices?

    SELECT 
        de.event_id, de.type, COUNT(*)
    FROM
    d.device_logs AS dl
        JOIN
    d.iot_devices AS id ON dl.device_id = id.device_id
        JOIN
    d.disaster_events AS de ON dl.timestamp BETWEEN DATE_SUB(de.timestamp,
        INTERVAL 5 MINUTE) AND de.timestamp
    WHERE
       (metric_type = 'temperature'
          AND value > 35)
          OR (metric_type = 'noise_level'
          AND value > 70)
          OR (metric_type = 'brightness'
          AND value < 10)
      GROUP BY 1 , 2


### 6)List devices that had more than 5 spikes in a day (Z-score logic approximation)?

    SELECT 
       device_id, DATE(timestamp), COUNT(*) AS a
    FROM
        d.device_logs
    WHERE
       (metric_type = 'temperature'
        AND value > 35)
        OR (metric_type = 'noise_level'
        AND value > 70)
        OR (metric_type = 'brightness'
        AND value < 10)
    GROUP BY 1 , 2
    HAVING a > 5

### 7)Find devices that repeatedly showed anomalies before multiple disasters?

    SELECT 
    dl.device_id, COUNT(DISTINCT event_id) AS d
    FROM
       d.device_logs AS dl
           JOIN
       d.disaster_events AS de ON dl.timestamp BETWEEN DATE_SUB(de.timestamp,
          INTERVAL 5 MINUTE) AND de.timestamp
    WHERE
       (metric_type = 'temperature'
          AND value > 35)
          OR (metric_type = 'noise_level'
          AND value > 70)
          OR (metric_type = 'brightness'
          AND value < 10)
    GROUP BY 1

## 🧠 Conclusion: Turning Device Anomalies into Actionable Insights
The analysis highlights a strong link between environmental anomalies and potential disaster events, with specific patterns in brightness, temperature, and sensor activity acting as early indicators. Key device types such as thermostats and noise sensors often report critical issues before disasters occur, enabling proactive detection.

To improve disaster preparedness and operational efficiency:

-Automated alerts for low brightness and rising temperature can flag power issues or heat stress.

-Recalibrating anomaly thresholds and investigating frequent anomaly sources can enhance detection accuracy.

-Predictive models trained on pre-disaster patterns can enable early-warning systems.

-Cross-sensor risk scoring helps triage disaster impact and prioritize response.

-Devices with frequent abnormal spikes should be targeted for calibration and closer monitoring, identifying potential failure points or high-risk zones.

-By combining these insights, organizations can shift from reactive disaster management to a data-driven, proactive prevention strategy, increasing safety, reliability, and system resilience.















