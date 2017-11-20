# ansible-beats-playbook

This playbook is for setting up version 6.x of Beats for shipping data to a remote Elasticsearch server:

- Metricbeat takes the metrics and statistics that it collects and ships them to the output that you specify, such as Elasticsearch or Logstash;

- Filebeat monitors the log directories or specific log files, tails the files, and forwards them either to Elasticsearch or Logstash for indexing.
