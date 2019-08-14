
# ZDOD19 - Monitoring in Azure 

Demo repository for my #ZDOD19 talk about Azure Monitoring

Steps:

1. Create the Function App
2. Explore Azure Monitoring

Dashboard:

![image](https://user-images.githubusercontent.com/365744/63044569-c63df380-bece-11e9-8f3c-77c456d772e2.png)


## Create the Function App

For now, I did this manually -- automated setup via AzureRM to follow...

### Azure Resources

Create Function App:

 * name: zdod19-app
 * OS: Windows
 * runtime: nodejs
 * plan: new app serice plan in S1 tier
 * with appinsights

Notes:
 * Linux apps can't be edited via Azure Portal currently
 * However, Windows apps don't support Diagnostic Logs :-/
 * Consumption based apps don't show metrics, don't support autoscale
 * Even with full App, metrics are only shown with B1 onwards (S1 for autoscale support)
 * neither of the available storage types supports Diagnostic Logs
 * client ip is not logged automatically by AppInisghts


### Add Endpoints

#### /hello

Start with a simple `/api/hello` endpoint:

```javascript
module.exports = async function (context, req) {

    if (req.query.name || (req.body && req.body.name)) {
        context.log('saying hello to ' + (req.query.name || req.body.name));

        // add artificial "processing" delay up to 2s...
        await new Promise(done => setTimeout(done, Math.random()*2000));

        context.res = {
            status: 200, body: "Hello " + (req.query.name || req.body.name)
        };
    }
    else {
        context.res = {
            status: 400, body: "Please pass a name on the query string or in the request body"
        };
    }
};
```

Try it out:
 * https://zdod19.azurewebsites.net/api/hello?name=yourname


#### /play

Next, let's add a more complex `/api/play` endpoint, which is a bit more CPU and memory intensive:

```javascript
// generates a random string between 0-7 chars
function randomString() {
    return Math.random().toString(36).substr(Math.floor(Math.random()*7));
}

module.exports = async function (context, req) {
    
    const ip = req.headers['client-ip'].split(':')[0];

    if (req.query.name) {

        let player = req.query.name;
        context.log("Player " + player + " with ip " + ip + " throws the dice...");

        // do some CPU heavy + memory consuming "gambling" to produce a score
        let difficulty = Math.random() * 7;
        let result = "";
        for (var i = Math.pow(difficulty, 7); i >= 0; i--) {		
		    result += randomString() + randomString() + randomString() + randomString();
	    };
        let score = result.length;

        // that weird bug that sometimes occurs...
        if (Math.random() > 0.999) {
            throw "I'm a Heisenbug!";
        }

        // success response
        context.log("Player " + player + " scored " + score + " with difficulty " + difficulty);
        context.res = {
            status: 200, body: "Hello " + player + ", you scored: " + new Intl.NumberFormat('en-US').format(score) + " (difficulty: " + Math.round(difficulty) + ")"
        };
    }
    else {
        // error response
        context.res = {
            status: 400, body: "Please pass a name on the query string"
        };
    }
};
```

Try it out:
 * https://zdod19.azurewebsites.net/api/play?name=yourname

#### /health

Let's also add a dedicated healthcheck endpoint for the availability tests:

```javascript
module.exports = async function (context, req) {

    // simulate occasional downtimes...
    if (Math.random() < 0.95) {
        context.res = { 
            status: 200, body: "I'm fine :-)"
        }
    } else {
        context.res = {
            status: 503, body: "Sorry I'm busy... :-("
        }
    }
};
```

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

### Add Metrics to Dashboard

### Add Metric Alert

## Explore AppInisghts

### Add AppInisghts to Dashboards
