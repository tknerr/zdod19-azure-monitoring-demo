
# #ZDOD19 - Monitoring in Azure 

Demo repository for my #ZDOD19 talk about Azure Monitoring: https://www.slideshare.net/tknerr/monitoring-in-azure

Overview:

1. Explore the Sample Function App
2. Explore Azure Monitoring

Teaser:

![image](https://user-images.githubusercontent.com/365744/63044569-c63df380-bece-11e9-8f3c-77c456d772e2.png)


## Explore the Sample Function App

The FunctionApp has been created up-front, see [FUNCTIONAPP.md](./FUNCTIONAPP.md) on how to re-create it on your own.

Demo steps:

 * show AppServicePlan (hosting environment)
 * show FunctionApp
   * [/api/hello](https://zdod19.azurewebsites.net/api/hello?name=yourname)
   * [/api/play](https://zdod19.azurewebsites.net/api/play?name=yourname)
   * [/api/health](https://zdod19.azurewebsites.net/api/health)
 * let's create some data!


## Explore Azure Monitor Components

We will look at different components [Azure Monitor](https://docs.microsoft.com/en-us/azure/azure-monitor/overview) consists of:

 * Metrics
 * Application Inisghts
   * Availability Tests
   * Live Metrics
   * Logs
 * Dashboards
 * Alerts
 * Service Health

### Explore Metrics

Demo steps:

 * show metrics on AppServicePlan level, pin to dashboard
    * avg vs max CPU
    * avg vs max Memory
 * show metrics on AppService level, pin to dashboard
    * HTTP 2xx vs 4xx vs 5xx requests
 * create metric alert for high CPU avg

### Explore AppInisghts

Demo steps:

 * show appInsights overview
 * show health checks, pin them to dashboard
 * show live metrics, pin to dashboard
 * show logs
    * requests
    * traces
    * search
    * performanceCounters
    * availabilityResults
 * create alert for chucknorris

For more examples, see [APPINSIGHTS.md](./APPINSIGHTS.md)

### Explore Dashboards

Import `dashboard.json` and explore the full featured dashboard:

 * show import / export
 * show time chooser
 * show queries behind appinisghts widgets

### Explore Service Health

Demo steps:

 * show Service Health / Health History
 * show https://status.azure.com
 * show how to set up Service Health alerts!
