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

	`payment-service` telemetry and log records.
	`cpu_percent`, `memory_percent`, `log_level`, and `message`.
	response time and 80% CPU or memory utilization, and builds an `ANOMALY`
	event with the reasons and original record.
	`EventTopic`.
	that stores, returns, and clears messages.
	runs detection, publishes detected events, consumes events, and reports
the
	records processed, anomalies detected, and events consumed.


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

Task 3: Validate Anomaly Detection and Event Streaming

I used the provided AnomalyDetector with its default thresholds: response time
above 500 milliseconds, CPU above 80 percent, or memory above 80 percent. I
then ran the existing AIOps pipeline against data/service_data.json. The
pipeline processed all 10 observations, created anomaly events for the two
abnormal observations, published them through the existing EventProducer and
EventTopic components, and consumed both events successfully.

The first detected anomaly occurred at 10:05. Its response time was 610
milliseconds, its CPU and memory values were 75 and 70 percent, and its ERROR
log said Payment service timeout. It was flagged for high response time and a
concerning log event.

The second detected anomaly occurred at 10:06. Its response time was 640
milliseconds, CPU was 94 percent, and memory was 91 percent. Its ERROR log said
Database connection timeout. It was flagged for high response time, high CPU
utilization, high memory utilization, and a concerning log event.

The remaining eight observations were treated as normal. They had successful
INFO logs, response times between 120 and 150 milliseconds, CPU between 42 and
50 percent, and memory between 51 and 57 percent. No normal observation was
incorrectly flagged, and no expected anomaly in the supplied data was missed
after ERROR logs were included in the detection reasons.

During validation, I corrected two issues in the provided workflow without
changing its architecture. The detector now recognizes both WARNING and ERROR
levels as concerning log events. The consumer now reads from the same existing
topic used by the producer, so detected events are available to the streaming
workflow instead of being left on a different in-memory topic.

One limitation is that the detector uses fixed thresholds rather than learning
the service's normal range or considering trends over time. A gradual slowdown
that remains below a threshold could therefore be missed. A useful improvement
would be to compare recent observations with a service baseline while retaining
the current threshold checks.

Task 4: Verify the AIOps Event Flow

I ran the provided workflow with data/service_data.json. It processed 10
records, detected 2 anomalies, and consumed 2 events. The execution printed
both received anomaly events, including their service, timestamp, type, and
reasons.

The Event is the anomaly message created by AnomalyDetector when an observation
has abnormal metrics or a concerning log. The Producer is EventProducer. It
receives the detected event and passes it to the Topic. The Topic is the
in-memory EventTopic named service-events, which stores the published event.
The Consumer is EventConsumer, which reads the event from that same topic.
The downstream AIOps component is run_pipeline, which collects the consumed
event in its events_consumed result and prints the event details in the final
pipeline report.

I also verified the individual handoff using the 10:05 payment-service anomaly.
Detection produced an event, EventProducer.publish returned true, the event
was stored on service-events, and EventConsumer.consume returned that event.
This confirms that the anomaly travelled through detection, production, the
topic, consumption, and the downstream AIOps result.

The verified flow was: AnomalyDetector created the event for the 10:05 timeout,
EventProducer received and published it, EventTopic stored it, and
EventConsumer received and processed it. The consumed event then returned to
run_pipeline as the downstream AIOps result. The complete run finished with 2
anomalies detected and 2 events consumed.

telemetry can be analyzed automatically, converted into actionable anomaly
events, and passed through a simple event-driven processing workflow. The
pipeline intentionally includes assessment issues around event-topic wiring
and log-level handling for investigation.


Task 5: Investigate and Correct the Workflow

The first problem was in src/anomaly_detector.py. The detector checked for a
WARNING log level, but the supplied operational data uses ERROR for both timeout
events. Because of this mismatch, the detector did not include the log event as
a reason for an anomaly. I corrected the condition so that both WARNING and
ERROR are treated as concerning log events. I ran the pipeline again and the
10:05 and 10:06 events were both reported with the reason Concerning log event.

The second problem was in src/aiops_pipeline.py. EventProducer published to the
service-events topic, while EventConsumer was connected to a separate
anomaly-events topic. The consumer therefore received zero events even though
the detector found anomalies. I corrected the workflow by connecting the
consumer to the existing producer topic. I ran the pipeline again and verified
that the two detected events were published and consumed successfully.

The final verification processed 10 records, detected 2 anomalies, and consumed
2 events. The reported anomalies were the payment service timeout at 10:05 and
the database connection timeout at 10:06. Their metric values, log messages,
timestamps, and detection reasons were included in the output. The corrections
use the existing detector, producer, topic, consumer, and pipeline components;
no unrelated event-processing implementation was introduced.

---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

