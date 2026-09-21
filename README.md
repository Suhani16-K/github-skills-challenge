# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!


---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)


task 2

Service monitored: payment-service
Data type: minute-level operational events
Metrics: response_time_ms, cpu_percent, memory_percent
Log details: log_level, message
Normal observations: stable response time, low CPU/memory, INFO messages
Unusual observations: ERROR messages and large spikes in latency/resource usage at 10:05 and 10:06


task 3

Anomaly Detection and Event Validation
The project uses the provided anomaly detection component in anomaly_detector.py to scan the operational records in service_data.json. The detector evaluates the service telemetry and flags any record that exceeds the configured thresholds for response time, CPU utilization, or memory utilization. It then returns an anomaly event containing the timestamp, service name, anomaly type, and the reasons for the flag.

The detector successfully processes the available operational data and identifies abnormal metric behavior. The clearest anomalies occur at 2026-09-20T10:05:00 and 2026-09-20T10:06:00. In these records, the response time rises to 610 ms and 640 ms, CPU usage reaches 75% and 94%, and memory usage reaches 70% and 91%. These values are well above the normal operating range and align with the ERROR-level log messages: “Payment service timeout” and “Database connection timeout”.

The detector distinguishes normal observations from abnormal observations by returning None for normal records and an ANOMALY event for high-risk records. Normal records such as 2026-09-20T10:00:00 through 2026-09-20T10:04:00 and 2026-09-20T10:07:00 through 2026-09-20T10:09:00 show stable response times, moderate CPU and memory levels, and INFO messages indicating successful payment processing. These are not flagged as anomalies.

The generated anomaly report is readable and contains enough information to explain why an observation was marked abnormal. Each anomaly includes the timestamp, service, anomaly type, and a list of reasons such as “High response time”, “High CPU utilization”, and “High memory utilization”. The log event is also relevant because the timeout messages confirm that the service is experiencing degradation.

The expected anomalies are detected using the provided thresholds, and no normal event in the steady-state data appears to be incorrectly flagged based on the metric values alone. However, the log rule in the existing detector is not aligned with the dataset because it checks for WARNING rather than ERROR, which is a limitation in the log-based detection logic. A possible improvement would be to use adaptive thresholds or a rolling baseline instead of fixed values, and to check the actual log severity used by the service so the detector better matches the real operational data.

task 4
Event Flow Validation
The repository contains a lightweight event-streaming simulation that demonstrates how a detected anomaly is moved through the pipeline. In this workflow, the anomaly detector creates an anomaly event when a record exceeds the configured thresholds. That event is then passed to the producer, which publishes it to the appropriate topic. The topic acts as the in-memory message channel, storing the event until it is consumed. The consumer reads the message from the topic and processes it as part of the downstream AIOps workflow.

The producer is responsible for taking an anomaly event and publishing it to the topic. The topic stores the event in memory so that it can be retrieved by the consumer. The consumer then receives the event from the topic and processes it for downstream handling. In this project, the event is a dictionary containing the anomaly metadata, including the timestamp, service name, type, and reason list. This allows the downstream AIOps component to understand the issue and react accordingly.

The event flow was validated by running the provided workflow against the sample operational data. The detector identified abnormal records, an event was produced, the producer published it to the topic, and the consumer retrieved the message from that topic. The final result confirms that the anomaly can travel through the complete event-processing pipeline and reach downstream AIOps processing.

Task 5: Investigate and Correct the Workflow


The main workflow problems were a mismatched topic configuration in aiops_pipeline.py and a log-severity mismatch in anomaly_detector.py. The producer and consumer were not using the same event topic, and the detector was checking for WARNING logs instead of the ERROR logs present in the operational data. These were corrected without changing the architecture so the anomaly events could be published, consumed, and processed correctly.


task 6
Execute the End-to-End Pipeline

After fixing the workflow issues, the complete AIOps pipeline was executed end to end using the provided architecture. The execution begins with the operational data in service_data.json, which is processed record by record. The anomaly detector inspects each record and identifies abnormal behavior based on the configured telemetry thresholds and relevant log severity.

Once an anomaly is detected, an anomaly event is generated. That event is passed to the producer, which publishes it to the in-memory event topic. The topic stores the event, and the consumer retrieves the event from the topic and processes it. The downstream AIOps component then receives the processed event and represents the identified operational issue in a readable final output.

The final output confirms that the workflow completed successfully:

operational data was processed
anomalous behavior was detected
an anomaly event was generated
the event was published to the topic
the event was consumed from the topic
the event was processed downstream
the final result reflects the service degradation and timeout condition