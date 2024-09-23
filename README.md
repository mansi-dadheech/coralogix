# Coralogix Task
---

## Prometheus Setup

### 1. Prometheus Configuration for integrating with Coralogix

I have used Centos Virtual machine as I don't have cloud setup available. 
For setting up Prometheus on a VM and integrating it with Coralogix, have made changes in `prometheus.yml` file contains the `remote_write` configuration as follows:

```yaml
remote_write:
- url: https://ingress.coralogix.in/prometheus/v1
  name: 'promtest'
  remote_timeout: 120s
  bearer_token: <API_KEY>
```
[Created a separate key for prometheus]
</br>

![image](https://github.com/user-attachments/assets/5ca91dfc-c510-465f-87d2-a882462c54b2)

</br>

After that, I was able to see plots on grafana.

## Metrics Queries and Dashboard
### 1. Number of CPUs on Node
```yaml
count(node_cpu_seconds_total{mode="system"}) by (instance)
```
### 2. Node Information
```yaml
max by (instance, nodename) (
  node_uname_info
  * on(instance) group_left(uptime_hours)
  (((time() - node_boot_time_seconds) / 3600) > 0)
  * on(instance) group_left(connections)
 node_netstat_Tcp_CurrEstab
)
```
### 3. Network Bandwidth
```yaml
sum(rate(node_network_transmit_bytes_total{device!="lo"}[5m])) by (instance)
```

### 4. Memory Information
```yaml
100 * (node_memory_MemTotal_bytes - node_memory_MemFree_bytes - node_memory_Cached_bytes - node_memory_Buffers_bytes) / node_memory_MemTotal_bytes
```
</br>

![image](https://github.com/user-attachments/assets/5fa25d7b-b571-41ce-97c6-54fc9dea04bd)

</br>

## Metrics Based Alerts
</br>
</br>

![image](https://github.com/user-attachments/assets/6965ed3d-f696-4229-a516-ae51197651d7)

</br>

## Log Parsing Using Regex
```yaml
(?<timestamp>\d{4}-\d{2}-\d{2} \d{2}.\d{2}.\d{2}\s(?:AM|PM)) (?<devicehost>\w+-\w+) - (?<status>\w+) - (?<message>.*)
```
</br>

![image](https://github.com/user-attachments/assets/852e85d2-2a6d-439a-a202-b89699134a88)

</br>

## Event2Metrics and Custom Dashboard
## Coralogix Event2Metric and Custom Dashboard Setup

I followed the **Coralogix Event2Metric** tutorial to create a metric named `mansi_authentication_metric`. The goal was to extract the following labels from the log entries:
- **USER**: The username attempting to authenticate.
- **STATUS**: Whether the authentication attempt was a success or failure.
- **DEVICEHOST**: The device from which the user attempted to connect.

Although I understood the problem and followed the steps in the Coralogix guide, the setup is not working as expected. I am unsure what is causing the issue.
</br>

![image](https://github.com/user-attachments/assets/2cf71e10-c0d4-4e8c-9e6f-474303af8720)

</br>


### 9. Custom Dashboard Creation

I also attempted to create a custom dashboard to display the following:

#### a. Top 5 Users by IP Location and Device Name
But the above values are not getting extracted so I was not able to do it. 
The idea was to use the Event2Metric data to display a table showing the top 5 users by:
- **IP Location**: Extracted from the IP address.
- **Device Name**: The device from which the user connected.
The query can be:
```yaml
count by(user, devicehost) (authentication_metric{status="success"})
```

#### b. Success Ratio Trendline
I tried to calculate the success ratio using:
```yaml
100 * (count_over_time(authentication_metric{status="failure"}[1h]) / 
       count_over_time(authentication_metric{status="success"}[1h]))
```










