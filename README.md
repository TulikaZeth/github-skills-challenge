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
	runs detection, publishes detected events, consumes events, and reports
the
	records processed, anomalies detected, and events consumed.

The purpose of AIOps in this assessment is to demonstrate how operational
telemetry can be analyzed automatically, converted into actionable anomaly
events, and passed through a simple event-driven processing workflow. The
pipeline intentionally includes assessment issues around event-topic wiring
and log-level handling for investigation.

Task 2: Analyse Logs and Metrics

The operational data contains three metric fields: response_time_ms records how
long a request took in milliseconds, cpu_percent records CPU usage, and
memory_percent records memory usage. The service field identifies the source of
each observation, but it is not itself a metric.

The log information is represented by log_level and message. The normal records
use the INFO level and say that the payment request was processed successfully.
The unusual records use the ERROR level and describe either a payment service
timeout or a database connection timeout.

The timestamp field records when each observation was made. The values use an
ISO-style date and time and cover 20 September 2026 from 10:00 through 10:09.
There is one observation per minute, so the timestamps provide the event order
and make it possible to compare the metrics and logs from the same moment. The
timestamps do not include a timezone.

The observations from 10:00 through 10:04 and from 10:07 through 10:09 appear
normal. Their response times stay between 120 and 150 milliseconds, CPU usage
stays between 42 and 50 percent, memory usage stays between 51 and 57 percent,
and every record reports a successful payment at INFO level.

The observation at 10:05 is unusual because the response time rises to 610
milliseconds and the log reports a payment service timeout at ERROR level. CPU
and memory are still moderate at 75 and 70 percent. The observation at 10:06 is
more severe: response time reaches 640 milliseconds, CPU reaches 94 percent,
memory reaches 91 percent, and the log reports a database connection timeout.
These two adjacent records indicate a short-lived service or database problem,
followed by a return to normal behaviour at 10:07.

---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

