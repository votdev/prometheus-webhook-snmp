prometheus-webhook-snmp is a Prometheus Alertmanager receiver that translates incoming notifications into SNMP traps.

# Features

- Independent from specific SNMP OID's
- The SNMP OID's of variable binds can be customized via prefix
- Extract SNMP OID from alert label
- Use a default SNMP OID for all alerts without an OID label
- Export Prometheus metrics from this receiver

# Start the receiver

To start the receiver execute the command:

    $ ./prometheus-webhook-snmp run

# Send a test SNMP trap

If you want to send a test SNMP trap, then simply execute the following command. This can be used to test your command line parameters.

    $ ./prometheus-webhook-snmp test

# Command line parameters

## Global

### --snmp-host
The host (IP or FQDN) of the SNMP trap receiver. Defaults to ``localhost``.

### --snmp-port
The port of the SNMP trap receiver. Defaults to ``162``.

### --snmp-community
The SNMP community string. Defaults to ``public``.

### --snmp-retries
Maximum number of request retries. Defaults to ``5``.

### --snmp-timeout
Response timeout in seconds. Defaults to ``1``.

### --alert-oid-label
The label where to find the OID. Defaults to ``oid``.

Example Prometheus alert rule:

```yaml
groups:
- name: test
  rules:
  - alert: load_0
    expr: node_load1 > 0
    for: 10s
    labels:
      severity: info
      oid: "1.3.6.1.4.1.7057.12.1.2.2"
    annotations:
      summary: "Instance {{ $labels.instance }} load is over 0!"
      description: "{{ $labels.instance }} of job {{ $labels.job }} load is over 0!"
```

### --trap-oid-prefix
The OID prefix for trap variable bindings. Defaults to ``1.3.6.1.4.1.7057.12``.

### --trap-default-oid
The trap OID if none is found in the Prometheus alert labels. Defaults to ``1.3.6.1.4.1.7057.12.1.2.1``.

## Command ``run``

### --host
The host to use, defaults to ``localhost``.

### --port
Port to listen for Prometheus Alertmanager notifications. Defaults to ``9099``.

### --metrics
Provide Prometheus metrics from this receiver under the URL ``/metrics``.

Example Prometheus configuration:

```yaml
scrape_configs:
  - job_name: 'prometheus-webhook-snmp'
    scrape_interval: 30s
    static_configs:
      - targets: ['localhost:9099']
```

# SNMP schema

## Traps

| OID | Description |
| :--- | :--- |
| ``trap-oid-prefix``.1.2.1 | The default trap. This is used if no OID is specified in the alert labels. |
| ``trap-oid-prefix``.1.2.[2...N] | Custom traps. |

## Objects

The following objects are appended as variable binds to a SNMP trap.

| OID | Type | Description |
| :--- | :---: | :--- |
| ``trap-oid-prefix``.1.1.1 | String | The status of the Prometheus alert. |
| ``trap-oid-prefix``.1.1.2 | String | The severity of the Prometheus alert. |
| ``trap-oid-prefix``.1.1.3 | String | Unique identifier for the Prometheus instance. |
| ``trap-oid-prefix``.1.1.4 | String | The name of the Prometheus job. |
| ``trap-oid-prefix``.1.1.5 | String | The Prometheus alert description field. |
| ``trap-oid-prefix``.1.1.6 | String | Additional Prometheus alert labels as JSON string. |
| ``trap-oid-prefix``.1.1.7 | Unix timestamp | The time when the Prometheus alert occurred. |
| ``trap-oid-prefix``.1.1.8 | String | The raw Prometheus alert as JSON string. |