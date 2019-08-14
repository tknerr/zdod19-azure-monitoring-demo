
# Sample AppInsights Queries

## Requests

### Find failed requests

```
requests
| where success == false 
```

### Requests by resultCode

```
requests
| summarize count() by tostring(customDimensions.HttpMethod), tostring(customDimensions.HttpPath), resultCode
```

## Performance Counters and Metrics

### CPU usage

```
performanceCounters
| where name == '% Processor Time'
| summarize max(value) by bin(timestamp, 1m)
| render timechart
```

### Free memory in GB

```
performanceCounters
| where name == 'Available Bytes'
| extend gb_value = value / 1024 / 1024 / 1024
| summarize min(gb_value) by bin(timestamp, 1m)
| render timechart
```

### Show which custom metrics are logged

```
customMetrics
| summarize count() by name, tostring(customDimensions.Category)
```


## Search

### Complete Trace for one request

Complete trace for a request (alternatively via Azure Portal):

```
search *
| where operation_Id == "f0af15f69151b74c8a0c3b98627f501e"
| order by timestamp desc 
```

### Search for a keyword appearing anywhere

```
search "pinger"
```

## Healthchecks

### Average duration for availability results / health checks

```
availabilityResults
| where timestamp > ago(1d) 
| summarize avg(duration) by location, bin(timestamp, 1m)
| render timechart 
```

## Real User Requests

### Only real user requests, ignoring healthchecks

```
requests 
| join kind=leftanti (availabilityResults) on operation_Id
```

### Avg duration of real user requests (per endpoint)

```
requests 
| join kind=leftanti (availabilityResults) on operation_Id
| extend ["call"]=strcat(tostring(customDimensions.HttpMethod), ' ', tostring(customDimensions.HttpPath))
| summarize avg(duration) by call, bin(timestamp, 1m) 
| render timechart 
```

### Number of real user requests (by status code)

```
requests 
| join kind=leftanti (availabilityResults) on operation_Id
| summarize count() by resultCode, bin(timestamp, 1m) 
| render timechart
```

## Logs / Traces

### User Logs only

```
traces
| where customDimensions.Category endswith '.User'
| order by timestamp desc
```

### Function host starts / stops

```
traces
| where customDimensions.Category contains 'Host.Startup'
| order by timestamp desc
```

### Latest user logs by log_level

```
traces
| where customDimensions.Category endswith '.User'
| extend log_level = tostring(customDimensions.LogLevel)
| project timestamp, message, log_level 
| order by log_level, timestamp desc 
| take 10
```

### Overview of all logs

```
traces
| extend log_level = tostring(customDimensions.LogLevel)
| extend log_category = tostring(customDimensions.Category)
| summarize log_count=count() by severityLevel, log_level, log_category
| order by severityLevel, log_count desc
| project log_level, log_category, log_count
```

## "Business" Metrics

### Plays by IP address

```
let msg_regex = "^Player (.*) with ip (.*) throws the dice";
traces
| where customDimensions.Category == 'Function.play.User'
| where customDimensions.Category endswith '.User'
| where message matches regex msg_regex
| extend player = extract(msg_regex, 1, message)
| extend ip = extract(msg_regex, 2, message)
| summarize count() by ip
| order by count_
```

### Top scorers

```
let msg_regex = "^Player (.*) scored (.*) with difficulty (.*)$";
traces
| where customDimensions.Category == 'Function.play.User'
| where message matches regex msg_regex
| extend player = extract(msg_regex, 1, message)
| extend score = toint(extract(msg_regex, 2, message))
| extend difficulty = toint(round(todecimal(extract(msg_regex, 3, message))))
| where difficulty > 0
| summarize max(score) by player, difficulty
| order by max_score, difficulty desc
```

### Max score by difficulty

```
let msg_regex = "^Player (.*) scored (.*) with difficulty (.*)$";
traces 
| where customDimensions.Category == 'Function.play.User'
| where message matches regex msg_regex
| extend score = toint(extract(msg_regex, 2, message))
| extend difficulty = toint(round(todecimal(extract(msg_regex, 3, message))))
| summarize max(score) by tostring(difficulty), bin(timestamp, 1m)
| render timechart 
```

### Max duration by difficulty

```
let msg_regex = "^Player (.*) scored (.*) with difficulty (.*)$";
traces 
| where customDimensions.Category == 'Function.play.User'
| join requests on operation_Id
| where message matches regex msg_regex
| extend score = toint(extract(msg_regex, 2, message))
| extend difficulty = toint(round(todecimal(extract(msg_regex, 3, message))))
| summarize max(duration) by tostring(difficulty), bin(timestamp, 1m)
| render timechart 
```