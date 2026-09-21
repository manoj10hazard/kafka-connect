Role Name
=========

A brief description of the role goes here.

Requirements
------------

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).

# Kafka Connect Ansible Role

Ansible role for installing and managing Apache Kafka Connect in distributed mode.

The role manages the following Kafka Connect plugins:

* Kafka Connect JDBC
* MySQL Connector/J
* Debezium PostgreSQL Connector
* Confluent Kafka Connect S3
* Confluent Avro Converter

The role is designed to be idempotent. Before installing a plugin, Ansible checks whether the required plugin already exists.

---

## Environment

| Component               | Version / Value              |
| ----------------------- | ---------------------------- |
| Kafka                   | 4.3.1                        |
| Kafka Connect           | 4.3.1                        |
| Kafka Connect Mode      | Distributed                  |
| Kafka Connect REST Port | 8083                         |
| Connect Cluster ID      | `connect-cluster`            |
| Plugin Directory        | `/opt/kafka/connect-plugins` |
| Kafka User              | `kafka`                      |
| Kafka Group             | `kafka`                      |
| Schema Registry         | 8.3.0                        |
| JDBC Connector          | 10.9.6                       |
| MySQL Connector/J       | 9.4.0                        |
| Debezium PostgreSQL     | 3.6.2.Final                  |
| S3 Connector            | 12.1.4                       |

---

## Role Structure

```text
kafka-connect/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── tasks/
│   ├── main.yml
│   └── plugins.yml
└── README.md
```

---

## Plugin Directory

All Kafka Connect plugins are installed under:

```text
/opt/kafka/connect-plugins/
```

Expected directory structure:

```text
/opt/kafka/connect-plugins/
├── kafka-connect-jdbc/
├── debezium-connector-postgres/
├── kafka-connect-s3/
└── avro-converter/
```

The MySQL Connector/J driver is installed inside the JDBC plugin directory:

```text
/opt/kafka/connect-plugins/kafka-connect-jdbc/
└── mysql-connector-j-9.4.0.jar
```

---

## Prerequisites

The target servers must have:

* Java installed
* Kafka installed
* Kafka Connect installed
* `kafka` user and group
* Network access to download plugins
* Required proxy configuration
* Ansible privilege escalation enabled

The playbook should use:

```yaml
become: true
```

Example:

```yaml
- name: Deploy Kafka Connect
  hosts: kafka_connect
  become: true

  roles:
    - kafka-connect
```

---

## Kafka Connect Configuration

Kafka Connect runs in distributed mode.

The Connect workers use the same Connect cluster ID:

```yaml
kafka_connect_group_id: "connect-cluster"
```

Internal Kafka topics:

```yaml
kafka_connect_config_storage_topic: "connect-configs"
kafka_connect_offset_storage_topic: "connect-offsets"
kafka_connect_status_storage_topic: "connect-status"
```

Replication factors:

```yaml
kafka_connect_config_storage_replication_factor: 3
kafka_connect_offset_storage_replication_factor: 3
kafka_connect_status_storage_replication_factor: 3
```

Partitions:

```yaml
kafka_connect_config_storage_partitions: 1
kafka_connect_offset_storage_partitions: 25
kafka_connect_status_storage_partitions: 5
```

---

## Kafka Bootstrap Servers

Bootstrap servers are configured using:

```yaml
kafka_connect_bootstrap_servers:
  - "k8s-lab-m1.wlink.com.np:9092"
  - "k8s-lab-w1.wlink.com.np:9092"
  - "k8s-lab-ww.wlink.com.np:9092"
```

These values should be changed according to the target Kafka cluster.

---

## Proxy Configuration

External plugin downloads use the configured proxy.

Example:

```yaml
PROXY_ENV:
  HTTP_PROXY: "http://http-proxy.wlink.com.np:5178"
  HTTPS_PROXY: "http://http-proxy.wlink.com.np:5178"
  http_proxy: "http://http-proxy.wlink.com.np:5178"
  https_proxy: "http://http-proxy.wlink.com.np:5178"
  no_proxy: ".wlink.com.np,localhost"
  NO_PROXY: ".wlink.com.np,localhost"
```

---

# Plugins

## 1. Kafka Connect JDBC

JDBC Connector version:

```yaml
kafka_connect_jdbc_version: "10.9.6"
```

Plugin directory:

```yaml
kafka_connect_jdbc_plugin_dir: "{{ kafka_connect_plugin_path }}/kafka-connect-jdbc"
```

Download URL:

```yaml
kafka_connect_jdbc_download_url: "https://hub-downloads.confluent.io/api/plugins/confluentinc/kafka-connect-jdbc/versions/{{ kafka_connect_jdbc_version }}/confluentinc-kafka-connect-jdbc-{{ kafka_connect_jdbc_version }}.zip"
```

The role:

1. Checks whether the JDBC connector is already installed.
2. Downloads the connector if it is missing.
3. Extracts the ZIP file.
4. Copies the connector libraries.
5. Sets ownership and permissions.
6. Restarts Kafka Connect when the plugin is installed.

---

## 2. MySQL Connector/J

MySQL Connector/J version:

```yaml
kafka_connect_mysql_driver_version: "9.4.0"
```

Download URL:

```yaml
kafka_connect_mysql_driver_download_url: "https://repo1.maven.org/maven2/com/mysql/mysql-connector-j/{{ kafka_connect_mysql_driver_version }}/mysql-connector-j-{{ kafka_connect_mysql_driver_version }}.jar"
```

The driver is installed inside:

```text
/opt/kafka/connect-plugins/kafka-connect-jdbc/
```

This allows the JDBC connector to connect to MySQL and MariaDB databases.

---

## 3. Debezium PostgreSQL Connector

Debezium version:

```yaml
debezium_version: "3.6.2.Final"
```

Plugin directory:

```yaml
debezium_dir: "{{ kafka_connect_plugin_path }}/debezium-connector-postgres"
```

The role downloads the Debezium PostgreSQL plugin from Maven Central.

Expected directory:

```text
/opt/kafka/connect-plugins/debezium-connector-postgres/
```

The role checks whether the connector already exists before downloading and extracting it.

---

## 4. Kafka Connect S3

S3 Connector version:

```yaml
s3_version: "12.1.4"
```

Plugin directory:

```yaml
s3_connector_dir: "{{ kafka_connect_plugin_path }}/kafka-connect-s3"
```

Download URL:

```yaml
s3_connector_download_url: "https://hub-downloads.confluent.io/api/plugins/confluentinc/kafka-connect-s3/versions/{{ s3_version }}/confluentinc-kafka-connect-s3-{{ s3_version }}.zip"
```

The role:

1. Checks whether the S3 connector is already installed.
2. Downloads the ZIP file if missing.
3. Extracts the ZIP file.
4. Copies the libraries from the `lib` directory.
5. Sets ownership and permissions.
6. Removes temporary files.
7. Restarts Kafka Connect when the plugin is installed.

---

## 5. Avro Converter

The Avro converter uses libraries provided by Schema Registry.

Schema Registry version:

```yaml
schema_registry_version: "8.3.0"
```

Schema Registry library directory:

```yaml
schema_registry_lib: "/opt/schema-registry/share/java"
```

Avro converter directory:

```yaml
avro_converter_dir: "{{ kafka_connect_plugin_path }}/avro-converter"
```

Required libraries:

```text
kafka-connect-avro-converter-<version>.jar
kafka-connect-avro-data-<version>.jar
kafka-avro-serializer-<version>.jar
kafka-schema-serializer-<version>.jar
```

The version is controlled by:

```yaml
schema_registry_version
```

For example:

```text
kafka-connect-avro-converter-8.3.0.jar
```

Before changing the version, verify the installed JARs:

```bash
ls -lh /opt/schema-registry/share/java/kafka-serde-tools/
```

---

# Idempotency

The role uses Ansible `stat` to check whether a plugin already exists.

For example:

```yaml
- name: Check Avro converter installation
  ansible.builtin.stat:
    path: "{{ avro_converter_dir }}/kafka-connect-avro-converter-{{ schema_registry_version }}.jar"
  register: avro_converter_installed
```

The installation task uses:

```yaml
when: not avro_converter_installed.stat.exists
```

This means:

```text
If the JAR exists:
    stat.exists = true
    not true = false
    installation task is skipped
```

If the JAR does not exist:

```text
If the JAR does not exist:
    stat.exists = false
    not false = true
    installation task runs
```

This prevents unnecessary downloads and installations when the playbook is run again.

---

# Permissions

Kafka Connect plugin files should be owned by:

```text
kafka:kafka
```

Plugin directories use:

```text
0750
```

Check ownership:

```bash
ls -ld /opt/kafka/connect-plugins/*
```

Check plugin files:

```bash
find /opt/kafka/connect-plugins -maxdepth 2 -type f -ls
```

Because plugin installation requires privileged operations, the playbook should use:

```yaml
become: true
```

at the play level.

---

# Running the Role

## Syntax Check

Always perform a syntax check first:

```bash
ansible-playbook -i inventory.ini site.yml --syntax-check
```

## Check Mode

Run Ansible in check mode:

```bash
ansible-playbook -i inventory.ini site.yml --check
```

## Deploy

Run the role:

```bash
ansible-playbook -i inventory.ini site.yml
```

## Run Only Plugin Tasks

The plugin tasks have the `plugins` tag.

Run:

```bash
ansible-playbook -i inventory.ini site.yml --tags plugins
```

---

# Verification

## Check Plugin Directories

```bash
ls -lh /opt/kafka/connect-plugins/
```

Expected:

```text
kafka-connect-jdbc
debezium-connector-postgres
kafka-connect-s3
avro-converter
```

## Check JDBC

```bash
ls -lh /opt/kafka/connect-plugins/kafka-connect-jdbc/
```

## Check Debezium

```bash
ls -lh /opt/kafka/connect-plugins/debezium-connector-postgres/
```

## Check S3

```bash
ls -lh /opt/kafka/connect-plugins/kafka-connect-s3/
```

## Check Avro

```bash
ls -lh /opt/kafka/connect-plugins/avro-converter/
```

---

# Kafka Connect Service Verification

Check the service:

```bash
systemctl status kafka-connect
```

Check port 8083:

```bash
ss -lntp | grep 8083
```

Test the REST API:

```bash
curl -s http://localhost:8083/ | jq
```

---

# Verify Connector Plugins

Use the Kafka Connect REST API:

```bash
curl -s http://localhost:8083/connector-plugins | jq
```

Expected connector types include:

```text
io.confluent.connect.jdbc.JdbcSourceConnector
io.confluent.connect.jdbc.JdbcSinkConnector
io.debezium.connector.postgresql.PostgresConnector
```

---

# Logs

Follow Kafka Connect logs:

```bash
journalctl -u kafka-connect -f
```

View recent logs:

```bash
journalctl -u kafka-connect --since "10 minutes ago"
```

---

# Updating Plugin Versions

Plugin versions are controlled through variables in:

```text
defaults/main.yml
```

Example:

```yaml
kafka_connect_jdbc_version: "10.9.6"
kafka_connect_mysql_driver_version: "9.4.0"
debezium_version: "3.6.2.Final"
s3_version: "12.1.4"
schema_registry_version: "8.3.0"
```

When changing a plugin version:

1. Update the version variable.
2. Verify the download URL.
3. Verify the expected JAR name.
4. Run `--syntax-check`.
5. Run the role.
6. Verify the Kafka Connect plugin list.

For Avro, verify the actual Schema Registry JARs first:

```bash
ls -lh /opt/schema-registry/share/java/kafka-serde-tools/
```

---

# Kafka Connect Cluster Requirement

Kafka Connect runs in distributed mode.

All Kafka Connect workers must use the same:

```yaml
kafka_connect_group_id: "connect-cluster"
```

and the same internal Kafka topics:

```text
connect-configs
connect-offsets
connect-status
```

All Kafka Connect workers must also have the required plugins installed.

For example, if there are three Kafka Connect workers:

```text
Worker 1
/opt/kafka/connect-plugins/

Worker 2
/opt/kafka/connect-plugins/

Worker 3
/opt/kafka/connect-plugins/
```

The plugin installation must therefore be performed on every Kafka Connect worker.

---

# Restart Behavior

Plugin installation tasks notify the Kafka Connect restart handler.

Kafka Connect is restarted only when a plugin installation task reports a change.

If no plugin changes are required, Kafka Connect is not unnecessarily restarted.

---

# Troubleshooting

Check Kafka Connect service:

```bash
systemctl status kafka-connect
```

Check Kafka Connect logs:

```bash
journalctl -u kafka-connect -f
```

Check port:

```bash
ss -lntp | grep 8083
```

Check plugin directory:

```bash
find /opt/kafka/connect-plugins -maxdepth 2 -type f | sort
```

Check Java:

```bash
java -version
```

Check Kafka Connect REST API:

```bash
curl -v http://localhost:8083/
```

Check installed connectors:

```bash
curl -s http://localhost:8083/connector-plugins | jq
```

---

# Author

Kafka Platform / Infrastructure Automation

# Status

Active development.

