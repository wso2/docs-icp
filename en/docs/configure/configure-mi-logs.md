# Monitor WSO2 Integrator: MI logs in WSO2 Integrator: ICP

WSO2 Integrator: ICP allows you to view MI logs with the help of OpenSearch and Fluent Bit. This guide will walk you through setting up Fluent Bit and OpenSearch for publishing Micro Integrator (MI) logs to the Integration Control Plane (ICP).

## Prerequisites
Before you begin, ensure you have the following prerequisites:

- **ICP server up and running**: If you haven't set up the ICP server yet, please refer to the [Quick Start Guide]({{base_path}}/get-started/quick-start-guide/) to set up the ICP server.
- **Integration project created in ICP**: If you haven't created a project yet, please refer to the [Create an Integration Project]({{base_path}}/get-started/create-project/) guide.
- **MI runtime installed and running**: You can download the WSO2 Integrator: MI Runtime from the [WSO2 website](https://wso2.com/integrator/).
- Docker and Docker Compose installed.

## Publishing WSO2 Integrator: MI logs to ICP

### Step 1: Setting up WSO2 Integrator: MI

1. Connect WSO2 Integrator: MI runtime to ICP. Please refer to the [Connecting WSO2 Integrator: MI Runtime to ICP]({{base_path}}/get-started/connect-integration/#connecting-wso2-integrator-mi-runtime-to-icp).

2. Modify the `CARBON_LOGFILE` log appender in the `log4j2.properties`.

    - Change the `layout.pattern` to match the ICP-supported log structure.

    {% raw %}
    ```properties
    appender.CARBON_LOGFILE.layout.pattern = [%d{yyyy-MM-dd'T'HH:mm:ss.SSSXXX}] %5p {%c} %X{Artifact-Container} - %m%ex ${sys:icp.runtime.log.suffix:-}%n
    ```
    {% endraw %}

    - Remove the `appender.CARBON_LOGFILE.fileName` config and change the `filePattern` config. This change is required to make sure Fluent Bit correctly detects daily rolling file updates.

    {% raw %}
    ```properties
    appender.CARBON_LOGFILE.filePattern = ${sys:logfiles.home}/wso2carbon-%d{MM-dd-yyyy}-%i.log
    ```
    {% endraw %}

    - Change the `strategy.type` to `DirectWriteRolloverStrategy`.

    {% raw %}
    ```properties
    appender.CARBON_LOGFILE.strategy.type = DirectWriteRolloverStrategy
    ```
    {% endraw %}


These changes will:

- Change the timestamp format to the ICP-supported format.
- Log the artifact container where applicable.
- Append the runtime id to the log.
- Change the rollover strategy and the file pattern to allow Fluent Bit to detect new log files created daily.

Optionally, if you want to keep the default configurations of the `CARBON_LOGFILE` appender as is or if there is a requirement to have the log file duplicated in a different location, you can add a new appender identical to the `CARBON_LOGFILE` in the `log4j2.properties` file.

{% raw %}
```properties
# Add the new appender to the list of appenders
appenders = CARBON_CONSOLE, CARBON_LOGFILE, MI_CARBON_LOGFILE,...

# Configure the new MI_CARBON_LOGFILE appender
appender.MI_CARBON_LOGFILE.type = RollingFile
appender.MI_CARBON_LOGFILE.name = MI_CARBON_LOGFILE
appender.MI_CARBON_LOGFILE.filePattern = <path/to>/wso2carbon-%d{MM-dd-yyyy}-%i.log
appender.MI_CARBON_LOGFILE.layout.type = PatternLayout
appender.MI_CARBON_LOGFILE.layout.pattern = [%d{yyyy-MM-dd'T'HH:mm:ss.SSSXXX}] %5p {%c} %X{Artifact-Container} - %m%ex ${sys:icp.runtime.log.suffix:-}%n
appender.MI_CARBON_LOGFILE.policies.type = Policies
appender.MI_CARBON_LOGFILE.policies.time.type = TimeBasedTriggeringPolicy
appender.MI_CARBON_LOGFILE.policies.time.interval = 1
appender.MI_CARBON_LOGFILE.policies.time.modulate = true
appender.MI_CARBON_LOGFILE.policies.size.type = SizeBasedTriggeringPolicy
appender.MI_CARBON_LOGFILE.policies.size.size=10MB
appender.MI_CARBON_LOGFILE.strategy.type = DirectWriteRolloverStrategy
appender.MI_CARBON_LOGFILE.filter.threshold.type = ThresholdFilter
appender.MI_CARBON_LOGFILE.filter.threshold.level = DEBUG

# Add an entry in the root loggers
rootLogger.level = INFO
rootLogger.appenderRef.CARBON_LOGFILE.ref = CARBON_LOGFILE
rootLogger.appenderRef.MI_CARBON_LOGFILE.ref = MI_CARBON_LOGFILE
...
```
{% endraw %}

### Step 2: Configure Fluent Bit and OpenSearch

#### 1. Environment Variables (.env)

Place the below environment variables in a `.env` file.

```env
OPENSEARCH_INITIAL_ADMIN_PASSWORD=<strong-password>
MI_LOG_FILE_PATH=/path/to/wso2mi/repository/logs
MI_LOG_FILE_NAME=wso2carbon-*.log
```

| Variable | Description |
|----------|-------------|
| `OPENSEARCH_INITIAL_ADMIN_PASSWORD` | Admin password for OpenSearch (min 8 chars, must include uppercase, lowercase, digit, and special character) |
| `MI_LOG_FILE_PATH` | Absolute path to the MI logs directory |
| `MI_LOG_FILE_NAME` | A wildcard pattern to match the `filePattern` in the MI `log4j2.properties` file, `CARBON_LOGFILE` log appender |

#### 2. Docker Compose Setup

Define the observability stack in a `docker-compose.yml` file.

```yaml
services:
  opensearch:
    image: opensearchproject/opensearch:3.5.0
    container_name: opensearch
    env_file:
      - path/to/.env
    environment:
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "OPENSEARCH_JAVA_OPTS=-Xms2g -Xmx2g"
      - "plugins.security.disabled=false"
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    ports:
      - "9200:9200"
      - "9600:9600"
    volumes:
      - opensearch_data:/usr/share/opensearch/data
    healthcheck:
      test:
        [
          "CMD-SHELL",
          "curl -k -f -u admin:${OPENSEARCH_INITIAL_ADMIN_PASSWORD} https://localhost:9200/_cluster/health?wait_for_status=yellow&timeout=10s || exit 1",
        ]
      interval: 15s
      timeout: 10s
      retries: 10
      start_period: 60s

  opensearch-dashboards:
    image: opensearchproject/opensearch-dashboards:3.5.0
    container_name: opensearch-dashboards
    ports:
      - "5601:5601"
    environment:
      - 'OPENSEARCH_HOSTS=["https://opensearch:9200"]'
      - "DISABLE_SECURITY_DASHBOARDS_PLUGIN=false"
    env_file:
    # For any environmental variables
      - path/to/.env
    volumes:
    # For any dashboard configurations
      - path/to/opensearch_dashboards.yml:/usr/share/opensearch-dashboards/config/opensearch_dashboards.yml
    depends_on:
      opensearch:
        condition: service_healthy

  fluent-bit:
    image: fluent/fluent-bit:4.2.2
    container_name: fluent-bit
    ports:
      - "2020:2020"
    env_file:
      - path/to/.env
    volumes:
      - path/to/fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf
      - path/to/parsers.conf:/fluent-bit/etc/parsers.conf
      - path/to/scripts:/fluent-bit/scripts
      - ${MI_LOG_FILE_PATH}:/var/log/mi:ro
      # Persist position database
      - fluent-bit-db:/fluent-bit/db
      # Persist buffer storage
      - fluent-bit-buffer:/fluent-bit/buffer
    # Wait for OpenSearch to be healthy and index templates to be created
    depends_on:
      opensearch:
        condition: service_healthy
      opensearch-setup:
        condition: service_completed_successfully

  opensearch-setup:
    image: curlimages/curl:8.18.0
    container_name: opensearch-setup
    env_file:
      - path/to/.env
    volumes:
    # Mount a setup directory containing the index template request file and dashboard templates
      - path/to/setup:/setup
    depends_on:
      opensearch-dashboards:
        condition: service_started
    command: >
      sh -c "
        echo 'Waiting for OpenSearch to be ready...' &&
        until curl -ku admin:$$OPENSEARCH_INITIAL_ADMIN_PASSWORD https://opensearch:9200/_cluster/health; do
          echo 'Waiting for OpenSearch...' &&
          sleep 10
        done &&

        echo 'Creating index templates...' &&
        curl -X PUT 'https://opensearch:9200/_index_template/wso2_integration_application_log_template' -ku admin:$$OPENSEARCH_INITIAL_ADMIN_PASSWORD -H 'Content-Type: application/json' -d @/setup/index-template-request.json &&

        echo 'Waiting for OpenSearch Dashboards to be ready...' &&
        until curl -f -u admin:$$OPENSEARCH_INITIAL_ADMIN_PASSWORD http://opensearch-dashboards:5601/api/status; do
          echo 'Waiting for OpenSearch Dashboards...' &&
          sleep 10
        done &&

        echo 'Importing dashboards...' &&
        curl -X POST 'http://opensearch-dashboards:5601/api/saved_objects/_import' -ku admin:$$OPENSEARCH_INITIAL_ADMIN_PASSWORD -H 'osd-xsrf: true' -F file=@/setup/opensearch-dashboards-template.ndjson &&

        echo 'Setup completed successfully!'
      "
volumes:
  opensearch_data:
  fluent-bit-db:
  # Size constrained by Fluent Bit storage.total_limit_size in fluent-bit.conf
  fluent-bit-buffer:
```

**Note:** It is important to ensure the index template is created before Fluent Bit starts sending logs, which guarantees correct field mappings.

#### 3. Fluent Bit Configuration

**Main Configuration (fluent-bit.conf)**

```ini
[SERVICE]
   Flush                     1
   Log_Level                 info
   Daemon                    off
   Parsers_File              parsers.conf
   # Enables monitoring endpoint at port 2020
   HTTP_Server               On
   HTTP_Listen               0.0.0.0
   HTTP_Port                 2020
   # Persistent storage path for buffered data
   storage.path              /fluent-bit/buffer
   storage.sync              normal
   storage.checksum          off
   storage.max_chunks_up     128
   # Memory limit for backlog data
   storage.backlog.mem_limit 5M
   # Hard limit on total disk usage for buffered data (prevents volume from filling)
   storage.total_limit_size  100M

# Input from Micro Integrator log files
[INPUT]
   Name                 tail
   Path                 /var/log/mi/${MI_LOG_FILE_NAME}
   Tag                  mi.*
   # Multiline handling
   multiline.parser     mi_multiline
   # Position tracking database (persistent)
   DB                  /fluent-bit/db/tail-mi.db
   DB.locking          true
   DB.journal_mode     WAL
   # File rotation handling
   Refresh_Interval    5
   Ignore_Older        2h
   # Buffer settings
   Buffer_Chunk_Size   512KB
   Buffer_Max_Size     2MB
   # Memory buffer limit per file
   Mem_Buf_Limit       256MB
   Skip_Long_Lines     On
   Read_from_Head      On
   # Adds the file path as a field in the record
   Path_Key          log_file_path

# Parse raw MI logs into structured fields using regex parser
[FILTER]
   Name    parser
   Match   mi.*
   Key_Name log
   Parser  mi_log_parser
   Reserve_Data On

# Enrich all MI logs with common fields
[FILTER]
   Name    lua
   Match   mi.*
   Script  /fluent-bit/scripts/scripts.lua
   Call    enrich_mi_logs

# This catches all mi.* logs and routes them to the correct output
[FILTER]
   Name rewrite_tag
   Match mi.*
   Rule $service_type ^MI$ mi_app_logs false
   Emitter_Name re_emitted_mi_app_logs

# Micro Integrator Application Logs
[OUTPUT]
   Name opensearch
   Host opensearch
   HTTP_User admin
   HTTP_Passwd ${OPENSEARCH_INITIAL_ADMIN_PASSWORD}
   # Creates daily indices (mi-application-logs-YYYY-MM-DD)
   Logstash_Format On
   Logstash_DateFormat %Y-%m-%d
   Logstash_Prefix mi-application-logs
   Match mi_app_logs
   # HTTP response buffer size (increase for large responses)
   Buffer_Size 2M
   Port 9200
   Replace_Dots On
   Suppress_Type_Name On
   tls On
   tls.verify Off
   Trace_Error On
```

**Parsers Configuration (parsers.conf)**

```ini
[PARSER]
    Name        mi_log_parser
    Format      regex
    Regex       ^\[(?<time>[^\]]+)\]\s+(?<level>\w+)\s+\{(?<class>[^}]+)\}\s+(?:\[\s*Deployed From Artifact Container:\s*(?<artifact_container>[^\]]+?)\s*\])?\s*-\s+(?<message>[\s\S]*)
    Time_Key    time
    Time_Format %Y-%m-%dT%H:%M:%S.%L%z
    Time_Keep   On

[MULTILINE_PARSER]
    Name          mi_multiline
    Type          regex
    Flush_timeout 1000
    Rule          "start_state"  "^\[[\d]{4}-[\d]{2}-[\d]{2}T[\d]{2}:[\d]{2}:[\d]{2}"  "cont"
    Rule          "cont"         "^(?!\[[\d]{4}-[\d]{2}-[\d]{2}T[\d]{2}:[\d]{2}:[\d]{2})"  "cont"    
```

This regex parser extracts the following fields from MI log lines:

| Field | Description | Example |
|-------|-------------|---------|
| `time` | Timestamp | `2026-02-09T10:30:45.123+0530` |
| `level` | Log level | `INFO`, `ERROR`, `WARN` |
| `class` | Java class name | `org.apache.synapse.api.API` |
| `artifact_container` | Deployed artifact name (optional) | `EmailService` |
| `message` | Log message content | `Initializing API: EmailService` |

**Lua Scripts (scripts.lua)**

```lua
function enrich_mi_logs(tag, timestamp, record)
    -- Set product identifier
    record["product"] = "Micro Integrator"
    record["service_type"] = "MI"

    -- Extract icp.runtimeId from message suffix and remove it
    if record["message"] then
        local runtimeId = string.match(record["message"], '%[icp%.runtimeId=([^%]]+)%]%s*$')
        if runtimeId then
            record["icp_runtimeId"] = runtimeId
            record["message"] = string.gsub(record["message"], '%s*%[icp%.runtimeId=[^%]]+%]%s*$', '')
        else
            record["icp_runtimeId"] = ""
        end
    else
        record["icp_runtimeId"] = ""
    end

    return 1, timestamp, record
end
```

The above script adds the below fields to each log record:

| Field | Description |
|-------|-------------|
| `product` | Set to "Micro Integrator" |
| `service_type` | Set to "MI" |
| `icp_runtimeId` | Extracted from message suffix `[icp.runtimeId=...]` |

#### 4. OpenSearch Index Template Configuration

The index template defines field mappings that are automatically applied to new indices matching the pattern `mi-application-logs-*`.

```json
{
    "index_patterns": ["mi-application-logs-*"],
    "template": {
        "mappings": {
            "properties": {
                "level": { "type": "keyword" },
                "time": { "type": "date" },
                "message": { "type": "text" },
                "product": { "type": "keyword" },
                "service_type": { "type": "keyword" },
                "class": { "type": "keyword" },
                "artifact_container": { "type": "keyword" },
                "icp_runtimeId": { "type": "keyword" }
            }
        }
    }
}
```

### Step 3: Start the Stack

```bash
# Start all services
docker compose -f path/to/docker-compose.yml --env-file path/to/.env up --build --force-recreate -d

# View logs
docker compose -f path/to/docker-compose.yml logs -f fluent-bit

# Stop all services
docker compose -f path/to/docker-compose.yml down
```

## Verifying the Setup

**1. Check Fluent Bit is watching MI logs**

```bash
docker compose -f docker-compose.yml logs fluent-bit | grep "inotify_fs_add"
```

**2. Check OpenSearch index exists**

```bash
curl -ku admin:<password> https://localhost:9200/_cat/indices/mi-application-logs-*
```

or log into the OpenSearch dashboards UI from [http://localhost:5601/](http://localhost:5601/) and follow the below path to check if the index exists:

```text
Management -> Index Management -> Indexes
```

**3. Query logs**

```bash
curl -X POST 'https://localhost:9200/mi-application-logs-*/_search' \
-ku admin:<password> \
-H 'Content-Type: application/json' \
-d '{
    "query": { "match_all": {} },
    "size": 10,
    "sort": [{"time": "desc"}]
}'
```

## Viewing MI logs on ICP

Once the Fluent Bit and OpenSearch has been properly set up, you can navigate to the logs tab of an ICP project or integration to see the published logs.

<a href="{{base_path}}/assets/img/configure/mi-logs/icp_mi_logs_view.png"><img src="{{base_path}}/assets/img/configure/mi-logs/icp_mi_logs_view.png" alt="MI logs in ICP" width="70%"></a>

## Troubleshooting

- Check Fluent Bit logs for errors:

```bash
docker compose -f docker-compose.observability.yml logs fluent-bit
```

- Make sure Fluent Bit is reading logs

```bash
curl http://localhost:2020/api/v1/metrics
```

- Check for buffer issues

If you see `cannot increase buffer` warnings, the `Buffer_Size` may need to be increased

- Check if Index template is correctly applied

If fields have wrong types assigned, delete existing indices and let them be recreated:
```bash
curl -X DELETE 'https://localhost:9200/mi-application-logs-*' -ku admin:<password>
```
