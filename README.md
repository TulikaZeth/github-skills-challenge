# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!

## AIOps Assessment

This repository contains a small event-driven AIOps pipeline for monitoring a
`payment-service`. The sample operational data in
`data/service_data.json` combines service metrics with application logs:
response time, CPU utilization, memory utilization, log level, and log message.

The operational problem is identifying degraded payment processing early. Most
requests complete normally, but timeout events can produce high response times,
CPU or memory utilization, and error logs. The pipeline turns those signals
into anomaly events so they can be handled as a stream instead of reviewed
manually one record at a time.

### Repository Components

- Operational data: `data/service_data.json` contains timestamped
	`payment-service` telemetry and log records.
- Metrics and logs: Each data record carries `response_time_ms`,
	`cpu_percent`, `memory_percent`, `log_level`, and `message`.
- Anomaly detection: `src/anomaly_detector.py` applies thresholds of 500 ms
	response time and 80% CPU or memory utilization, and builds an `ANOMALY`
	event with the reasons and original record.
- Event production: `src/event_producer.py` publishes detected events to an
	`EventTopic`.
- Event topics: `src/event_topic.py` provides the in-memory topic abstraction
	that stores, returns, and clears messages.
- Event consumption: `src/event_consumer.py` reads messages from a topic.
- Final AIOps processing: `src/aiops_pipeline.py` loads the JSON records,
	runs detection, publishes detected events, consumes events, and reports the
	records processed, anomalies detected, and events consumed.

The purpose of AIOps in this assessment is to demonstrate how operational
telemetry can be analyzed automatically, converted into actionable anomaly
events, and passed through a simple event-driven processing workflow. The
pipeline intentionally includes assessment issues around event-topic wiring
and log-level handling for investigation.


---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

