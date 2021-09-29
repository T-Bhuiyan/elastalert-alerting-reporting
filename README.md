
# sand-playground-for-elasticsearch-elastalert

ElastAlert is an alerting framework originally designed by Yelp. It is able to detect anomalies, spikes, or other patterns of interest. It is production-ready and is a well-known standard of alerting in the Elasticsearch ecosystem. It is designed to be reliable, highly modular, and easy to set up and configure.

ElastAlert configuration consists of three steps:

* Installing ElastAlert and its metadata indices.
* Configuring the main configuration file.
* Configuring the alert rules.

Elasticsearch is periodically queried and the data is passed to the rule type, which determines when a match is found. When a match occurs, it is given to one or more alerts, which take action based on the match. Several rule types with common monitoring paradigms are included with ElastAlert:

* "Matches when the value of a metric within the calculation window is higher or lower than a threshold" (metric aggregation type)
* “Match when the number of unique values for a field is above or below a threshold" (cardinality type)
* “Match when the rate of events increases or decreases” (spike type)
* “Match where there are at least X events in Y time" (frequency type)
* "Match on any event matching a given filter" (any type)

## Requirements:

* Elasticsearch
* ISO8601 or Unix timestamped data
* Python 3.6
* pip, see requirements.txt
* Packages on Ubuntu 14.x: python-pip python-dev libffi-dev libssl-dev

## Installation

You can either use the package manager [pip](https://pip.pypa.io/en/stable/) to install the latest released version of ElastAlert. But following steps will help you to install it properly without encountering error:

#### Python:

```bash
$ sudo apt update -y && sudo apt upgrade
$ sudo apt install software-properties-common
$ sudo add-apt-repository ppa:deadsnakes/ppa
$ sudo apt install -y python3
$ python3 -V
$ sudo apt install -y python3-pip
$ pip3 --version
$ sudo apt install -y build-essential libssl-dev libffi-dev python3-dev
```

#### ElastAlert:

```bash
$ cd elastalert
$ sudo pip3 install "setuptools>=11.3"
$ sudo pip3 install pyOpenSSL
$ sudo python3 setup.py install
$ sudo pip3 install "elasticsearch>=5.0.0"
$ cp config.example.yaml config.yaml
$ sudo vi config.yaml
```

## ElastAlert configuration file:

Edit *config.yaml* and update *es_host* with IP address or dns name of the elasticsearch server *es_port, buffer_time*.

* rules_folder: example_rules

```yaml
config.yaml

run_every:
  minutes: 1
buffer_time:
  hours: 1 
aggregation:
  schedule: 0 4 * * *
es_host: 127.0.0.1
es_port: 9200
use_ssl: False 
verify_certs: False 
es_username: <username>
es_password: <password>
writeback_index: elastalert_status
writeback_alias: elastalert_alerts
alert_time_limit:
  days: 2
```

* elastalert-create-index (create writeback indices in elasticsearch)

## Rules configuration

* This one is to show an example of Rules Type (*metric_aggregation*) for *Unique Streaming Session*

```yaml
example_streamingSessionID_agg.yaml

name: SAND_Reporting
type: metric_aggregation

#es_host: 127.0.0.1
#es_port: 9200

index: development-*
buffer_time:
hours: 1

metric_agg_key: streamingSessionID
metric_agg_type: cardinality

#query_key: beat.hostname
doc_type: _doc

#bucket_interval:
#minutes: 5
#sync_bucket_interval: true
#allow_buffer_time_overlap: true
#use_run_every_query_size: true

#min_threshold: 0.1
max_threshold: 0.1

aggregation:
     schedule: 0 4 * * *

alert: "ms_teams"
ms_teams:
ms_teams_webhook_url: "..."
ms_teams_alert_summary: "test"

alert_text: |
TOTAL_streamingSessionID: {0}

alert_text_type: alert_text_only
alert_text_args: ["metric_streamingSessionID_cardinality"]
```

#### MS Teams Integration

```
Please remember when setup MS Teams: (exclusive)
alert:
- "ms_teams" (not "ms teams")

ms_teams_alert_summary:
- "test"

That's also wrong, ms teams expect a string not a list
correct:

ms_teams_alert_summary: "test"
```

#### Test and Run

```bash
$ elastalert-test-rule example_rules/example_frequency.yaml
$ python3 -m elastalert.elastalert --verbose --rule example_frequency.yaml (running as a service)
```

#### Alert Notification Content Formatting

```yaml
alert_text: |
                 TOTAL streamingSessionID: {0}

alert_text_type: alert_text_only

alert_text_args: ["metric_streamingSessionID_cardinality"]
```

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License
