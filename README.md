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

The following fields represent metrics:

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


&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

