# WSO2 API LOGS

Data Source is Loki
Promtail is configured to collect logs from the wso2 api.log

wso2 api.log is enabled per API with the REST API from apictl

## Enable WSO2 API Logs

First, login to your API Manager instance
```bash
apictl login -e dev -k
```

List the available APIs to get their IDs
```bash
apictl get apis -e dev -k 
```

Enable the api.log with the API ID
```bash
apictl set api-logging --api-id <id> --log-level full -e dev -k
```

## Configure Promtail to collect the logs from api.log file

>*****notice** The log file needs to be able to be accessed by promtail use the below command to do that

```bash
sudo setfacl -m u:promtail:--x /pathtoapilog
sudo setfacl -m u:promtail:--x /pathtoapilog/api.log
```

The parent directories of the file must also be allowed to be accessed by promtail user account.

## Promtail Config

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
- url: http://grafana.intmm.net:3100/loki/api/v1/push

scrape_configs:
- job_name: wso2api
  static_configs:
  - targets:
      - localhost
    labels:
      job: api.log
      #NOTE: Need to be modified to scrape any additional logs of the system.
      __path__: /path/to/api.log
   
  pipeline_stages:
  - regex:
      expression: '^\[(?P<log_time>[^\]]+)\].* - (?P<json_data>\{.*\})'
  - json:
      source: json_data
      expressions:
        headers: headers
        payload: payload
        correlationId: correlationId
        verb: verb
        apiTo: apiTo
        flow: flow
        statusCode: statusCode
  - labels:
      log_time:  
      correlationId:
      verb:
      apiTo:
      flow:
      statusCode:
  - output:
      source: json_data
``` 

