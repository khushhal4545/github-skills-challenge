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

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

