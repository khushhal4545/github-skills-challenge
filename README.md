# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!

## AIOps Assessment Scenario

This exercise monitors a `payment-service` that processes payment requests. The
operational issue is intermittent service degradation: response times can become
high, CPU and memory utilization can spike, and payment or database timeouts can
appear in the logs. These symptoms can affect payment processing and require
quick detection.

AIOps is used here to combine operational telemetry and log information,
identify anomalous records, and publish anomaly events for downstream
processing. The assessment demonstrates a simple monitoring workflow before
more advanced alerting or remediation is added.

## Repository Components

- `data/service_data.json`: Operational data containing service metrics and logs.
- `src/anomaly_detector.py`: Applies thresholds to response time, CPU, and memory
	values and creates anomaly events.
- `src/event_producer.py`: Publishes detected anomaly events.
- `src/event_topic.py`: Provides the in-memory event topic used to store events.
- `src/event_consumer.py`: Reads events from an event topic.
- `src/aiops_pipeline.py`: Runs the complete flow: loads data, detects anomalies,
	publishes events, and consumes the results.
- `src/calculations.py`: Standalone example utility module used by its tests; it
	is not part of the AIOps workflow.
- `tests/`: Automated tests for calculations and the AIOps event pipeline.


---


## Task 2: Analyse Logs and Metrics 

### 1. Fields Representing Metrics

The following fields represent metrics:-

* `response_time_ms` – response time of the payment service in milliseconds.
* `cpu_percent` – CPU utilization percentage.
* `memory_percent` – memory utilization percentage.

### 2. Fields Representing Log Information

The following fields represent log information:

* `log_level` – indicates the log severity, such as `INFO` or `ERROR`.
* `message` – describes the event or activity that occurred.

The `service` field identifies the service generating the observation.

### 3. Use of Timestamps

The `timestamp` field records the time at which each observation occurred.

The data contains observations from `2026-09-20T10:00:00` to `2026-09-20T10:09:00`, with one observation recorded every minute.

Timestamps allow the operational data to be analysed chronologically and help identify when unusual behaviour occurred.

### 4. Observations Representing Normal Behaviour

The observations from **10:00 to 10:04** and **10:07 to 10:09** appear to represent normal behaviour.

During these periods:

* Response time is approximately 120–150 ms.
* CPU utilization is approximately 42–50%.
* Memory utilization is approximately 51–57%.
* Log level is `INFO`.
* Payment requests are reported as processed successfully.

The metrics remain relatively stable during these periods.

### 5. Observations Representing Unusual Behaviour

The observations at **10:05 and 10:06** appear to represent unusual behaviour.

At **10:05**:

* Response time increased to `610 ms`.
* CPU utilization increased to `75%`.
* Memory utilization increased to `70%`.
* Log level changed to `ERROR`.
* Message: `Payment service timeout`.

At **10:06**:

* Response time increased to `640 ms`.
* CPU utilization increased to `94%`.
* Memory utilization increased to `91%`.
* Log level was `ERROR`.
* Message: `Database connection timeout`.

These observations are significantly different from the surrounding observations. The increase in response time and resource utilization, together with the `ERROR` logs, indicates abnormal behaviour.

At **10:07**, the metrics returned to approximately their previous range and the log level returned to `INFO`, suggesting that the unusual behaviour was temporary.

### Summary

The `payment-service` shows mostly stable behaviour, with a short period of unusual activity between **10:05 and 10:06**. This period is identified by significantly higher response time, CPU and memory utilization, along with `ERROR` log messages related to payment and database timeouts.



## Task 5: Investigate and Correct the Workflow

### Issue 1: Incorrect Log-Level Detection

- **Affected component:** `src/anomaly_detector.py`
- **Cause:** The detector treated `WARNING` log entries as errors.
- **Correction:** Changed the condition from `WARNING` to `ERROR`.
- **Verification:** The pipeline correctly identifies `ERROR` log events as anomalies.

### Issue 2: Producer and Consumer Used Different Topics

- **Affected component:** `src/aiops_pipeline.py`
- **Cause:** The producer published events to `service-events`, while the consumer was listening to `anomaly-events`.
- **Correction:** Configured the consumer to use the same `producer_topic`.
- **Verification:** After the correction, the pipeline consumed both detected anomaly events.

### Execution Result

```text
Records processed: 10
Anomalies detected: 2
Events consumed: 2


#Task 3: Identify Anomalies

The provided AnomalyDetector was used to analyse the operational data.

The detector uses the following thresholds:

Response time threshold: 500 ms
CPU threshold: 80%
Memory threshold: 80%
Error log condition: ERROR

An observation is returned as an anomaly when one or more of these conditions are met.

Detection Findings

Two anomalous observations were detected.

Anomaly 1
Service: payment-service
Timestamp: 2026-09-20T10:05:00
Type: ANOMALY

Reasons:

High response time
Error log detected

Source information:

Response time: 610 ms
CPU: 75%
Memory: 70%
Log level: ERROR
Message: Payment service timeout
Anomaly 2
Service: payment-service
Timestamp: 2026-09-20T10:06:00
Type: ANOMALY

Reasons:

High response time
High CPU utilization
High memory utilization
Error log detected

Source information:

Response time: 640 ms
CPU: 94%
Memory: 91%
Log level: ERROR
Message: Database connection timeout

The remaining observations were treated as normal because they did not cross the configured thresholds and did not contain an ERROR log.



#Task 4: Verify the AIOps Event Flow

The provided event-streaming components were used to verify that anomaly events can travel through the complete pipeline.

Event Processing Flow
Operational Data
        ↓
Anomaly Detector
        ↓
Anomaly Event
        ↓
Event Producer
        ↓
service-events Topic
        ↓
Event Consumer
        ↓
Downstream AIOps Processing
Component Roles
Event / Message

An event contains the detected anomaly information, including:

Timestamp
Service
Event type
Reasons
Source record
Producer

The producer receives detected anomaly events and publishes them to the configured event topic.

Topic

The topic provides the in-memory communication channel used to pass events from the producer to the consumer.

Consumer

The consumer receives events from the topic and processes the detected anomaly events.

Downstream AIOps Component

The downstream part of the pipeline represents processing of the consumed anomaly event and provides the final operational issue information.

Verification

The initial execution showed:

Records processed: 10
Anomalies detected: 2
Events consumed: 0

This identified a problem in the event flow because anomalies were detected but were not being consumed.

After correcting the topic configuration, the pipeline produced:

Records processed: 10
Anomalies detected: 2
Events consumed: 2

This verified that both anomaly events successfully travelled through the event-processing pipeline.

Task 5: Investigate and Correct the Workflow

Two issues were identified and corrected while troubleshooting the provided architecture.

Issue 1: Incorrect Log-Level Detection
Affected Component

src/anomaly_detector.py

Cause

The detector incorrectly treated WARNING log entries as errors:

if record["log_level"] == "WARNING":
    reasons.append("Error log detected")

This did not correctly represent the assessment requirement because the concerning log events in the dataset were ERROR events.

Correction

The condition was changed to:

if record["log_level"] == "ERROR":
    reasons.append("Error log detected")
Verification

After the correction, the detector correctly identified the ERROR log events associated with the anomalous observations.

Issue 2: Producer and Consumer Used Different Topics
Affected Component

src/aiops_pipeline.py

Cause

The producer published events to:

service-events

while the consumer was listening to a different topic:

anomaly-events

Therefore, the producer generated anomaly events but the consumer could not receive them.

Correction

The existing architecture was retained and the consumer was configured to use the same topic as the producer:

producer_topic = EventTopic("service-events")

detector = AnomalyDetector()
producer = EventProducer(producer_topic)

consumer = EventConsumer(producer_topic)
Verification

Before the correction:

Anomalies detected: 2
Events consumed: 0

After the correction:

Anomalies detected: 2
Events consumed: 2

This confirmed that the topic configuration problem was resolved.

Task 6: Execute the End-to-End Pipeline

After completing the investigation and corrections, the complete AIOps workflow was executed.

Execution Command
python src/aiops_pipeline.py
Final Execution Result
==================================================
AIOps Pipeline Result
==================================================
Records processed: 10
Anomalies detected: 2
Events consumed: 2
Detected Events
Event 1
Service: payment-service
Timestamp: 2026-09-20T10:05:00
Type: ANOMALY
Reasons: High response time, Error log detected
Event 2
Service: payment-service
Timestamp: 2026-09-20T10:06:00
Type: ANOMALY
Reasons: High response time, High CPU utilization, High memory utilization, Error log detected
End-to-End Verification

The final execution verified all stages of the AIOps workflow:

Operational data was processed.
Abnormal behaviour was detected.
Anomaly events were generated.
The events were published by the producer.
The events were passed through the configured topic.
The events were consumed and processed.
The final output identified the operational issues affecting payment-service.

The final result was:

10 records processed
2 anomalies detected
2 events consumed
Task 7: Findings and Reproduction
Overall Findings

The AIOps simulation successfully demonstrates a basic operational monitoring and event-processing workflow.

The provided operational data showed mostly stable service behaviour with a short period of degradation at 10:05 and 10:06.

The main detected issues were:

Increased response time
Increased CPU utilization
Increased memory utilization
Payment service timeout
Database connection timeout

The anomaly detector successfully converted these abnormal observations into anomaly events.

The events were then published by the producer, passed through the in-memory topic, received by the consumer, and processed successfully.

Limitations and Possible Improvements

The current anomaly detection approach uses fixed thresholds:

Response time > 500 ms
CPU > 80%
Memory > 80%

Fixed thresholds may not adapt to different workloads or changing normal service behaviour.

A possible improvement would be to introduce dynamic or baseline-based thresholds that learn normal service behaviour over time.

Other possible improvements include:

Persistent event storage
More detailed alerting
Historical trend analysis
Additional anomaly-detection techniques
Automated remediation actions
More detailed monitoring dashboards
Reproduction Steps

Another user can reproduce the demonstration using the following steps.

1. Clone the repository
git clone <repository-url>
cd github-skills-challenge
2. Create a Python virtual environment
python -m venv .venv
3. Activate the virtual environment

On Linux/macOS:

source .venv/bin/activate

On Windows:

.venv\Scripts\activate
4. Install dependencies
pip install -r requirements.txt
5. Run the AIOps pipeline
python src/aiops_pipeline.py
6. Verify the expected result

The final output should show:

Records processed: 10
Anomalies detected: 2
Events consumed: 2

Conclusion

The completed assessment demonstrates a basic AIOps workflow in which operational telemetry is analysed, abnormal behaviour is detected, anomaly events are generated and published, and the events are subsequently consumed and processed.

The identified configuration and detection issues were corrected within the existing architecture, and the final end-to-end execution successfully processed the operational data and produced two anomaly events for payment-service.

The output should also display the detected anomaly events and their reasons.

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

