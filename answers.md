# Prerequisites - Setup the environment

I started by registering an account with DD and selected a few services I use:

[Screenshot: Services selection](https://drive.google.com/open?id=1LqQwx6UwDIf0dXYiFIFygJEhUXI5nUxX)

One of the steps during the registration process is actually installing the first DD Agent. I chose Docker for my first agent. If anything goes wrong during the agent installation, it should be easy to remove the agent ;-).

[Screenshot: Installation steps Docker Agent](https://drive.google.com/open?id=1FAsfBLTXc9U-ONCce0VsdATYhOwW_Ok7)

DD provided me with a one-step install which went through smoothly and I had my first agent reporting:

[Screenshot: dd-agent running in Docker](https://drive.google.com/open?id=1_0oTEcuj5w0AqCdOqUy0NH3SVI_BrzhA)

My first agent is reporting - only max. 5 min into the exercise:

[Screenshot: Docker Agent is reporting](https://drive.google.com/open?id=1qbLLBq66MxLTEC0uBBqiBVbDu1Zj_QSG)

I also chose to connect DD with my AWS account, which is an optional step during the registration process.

A few screenshot of the steps below:

[Screenshot: AWS Integration 1/3](https://drive.google.com/open?id=1iLrf-h7XjwWXO0RJq3NClT97r6ul7yKu)

[Screenshot: AWS Integration 2/3](https://drive.google.com/open?id=1UZc2uWOHWL1x217CktM_2vRYQlI1bEIh)

[Screenshot: AWS Integration 3/3](https://drive.google.com/open?id=1WfqC83K10rJMKBq9U9S2wV281YMZpllf)

And finally, my AWS account is integrated with DD:

[Screenshot: Confirmed AWS Integration](https://drive.google.com/open?id=1JIRDxAipf3-M2GMJDFEFVQWJb5NLGAcs)

# Collecting Metrics

Since I was already logged on to my AWS account, I chose to spin up an EC2 instance for this exercise and installed MySQL on it. Again, an easy one-step install and I had my second agent reporting:

[Screenshot: Amazon Linux integration](https://drive.google.com/open?id=1b-_1Af0_Ue_2OjraxrZmESzJ0BectyKE)

[Screenshot: Amazon Linux Agent installation](https://drive.google.com/open?id=1I954xlFd-pCOx1RD9MPHJu8DRM6KPzZm)

[Screenshot: MySQL Integration Installed](https://drive.google.com/open?id=1m-4bk-LTd4sHehtBv8t6UCYLIW5yELZk)

A DB user had to be created with the following grants:

[Screenshot: MySQL Grants given to DD](https://drive.google.com/open?id=1RJGWIbzfFx7RNNeY0M67Rtk7ewDZDKPg)

## Assigning tags

DD Documentation
  - [Starting with tagging](https://docs.datadoghq.com/tagging/)
  - [Assigning Tags](https://docs.datadoghq.com/tagging/assigning_tags/?tab=agentv6)

I edited the datadog.yaml file to add tags for my host:

```
tags:
    - env:aws
    - role:database
```

Tags were displayed on the host map:

[Screenshot: Host with new tags](https://drive.google.com/open?id=1D4cyM23QwKG1MihIdp5rHo56WDA8ph4x)

## Custom Agent check that submits a metric named my_metric with a random value between 0 and 1000.

File: /etc/datadog-agent/checks.d/hello.py
```
import random

# the following try/except block will make the custom check compatible with any Agent version
try:
    # first, try to import the base class from new versions of the Agent...
    from datadog_checks.base import AgentCheck
except ImportError:
    # ...if the above failed, the check is running in Agent version < 6.6.0
    from checks import AgentCheck

# content of the special variable __version__ will be shown in the Agent status page
__version__ = "1.0.0"


class HelloCheck(AgentCheck):
    def check(self, instance):
        self.gauge('hello.world', random.randint(0,1000))
```

With corresponding

File /etc/datadog-agent/conf.d/hello.yaml
```
instances: [{}]
```

My first metric hello.world is reporting:

[Screenshot: Metrics Explorer](https://drive.google.com/open?id=1q7U1Tx4sdaZQa2bY-k8abWYuDkg77RKv)

[Screenshot: Metrics graph](https://drive.google.com/open?id=1Ht_w8hQUDXOoWIvJDGrlQW0XROYHgCF2)

## Changing check's collection interval so that it only submits the metric once every 45 seconds.

DD Documentation
  - [Custom Agent check](https://docs.datadoghq.com/developers/write_agent_check/?tab=agentv6#collection-interval)

I added the parameter min_collection_interval in the configuration file to change the interval to 45:

File: /etc/datadog-agent/conf.d/hello.yaml
```
init_config:

instances:
  - min_collection_interval: 45
```

However, "it does not mean that the metric is collected every 45 seconds, but rather that it could be collected as often as every 45 seconds. The collector will try to run the check every 45 seconds but the check might need to wait in line, depending on how many integrations are enabled on the same Agent. Also if the check method takes more than 45 seconds to finish, the Agent will notice the check is still running and will skip its execution until the next interval."

## Bonus Question Can you change the collection interval without modifying the Python check file you created?

Yes, by changing the parameter min_collection_interval in the .yaml configuration file.

# Visualizing Data

DD Documentation
  - [Using Postman with Datadog APIs](https://docs.datadoghq.com/getting_started/api/)
  - [Anomaly Algorithm](https://docs.datadoghq.com/graphing/functions/algorithms/#anomalies)
  - [Anomaly FAQ](https://docs.datadoghq.com/monitors/faq/anomaly-monitor/#pagetitle)

Setting up the Postman environment:

[Screenshot: Postman environment](https://drive.google.com/open?id=1i-TXNS-nFtDugDBnEw3v77anfDGFXUZs)

Running an authentication check to see if it works:

[Screenshot: Authentication check](https://drive.google.com/open?id=1Ok1TIidjdz--rF-c3dYkOB0kK0Xj4GrQ)

## Utilize the Datadog API to create a Timeboard that contains:
- Your custom metric scoped over your host.
- Any metric from the Integration on your Database with the anomaly function applied.
- Your custom metric with the rollup function applied to sum up all the points for the past hour into one bucket

The body for the API call to create the dashboard:

```
{
    "title": "My First Dashbord",
    "widgets": [
        {
            "definition": {
                "type": "timeseries",
                "requests": [
                    {
                        "q": "avg:hello.world{*}"
                    }
                ],
                "title": "Hello World timeseries"
            }
        },
        {
        	"definition": {
            	"type": "timeseries",
            	"requests": [
            		{
            			"q": "anomalies(avg:mysql.performance.cpu_time{*}, 'basic', 2)"
            		}
            	],
        		"title": "MySQL Anomaly Graph cpu_time"
			}
        },
        {
            "definition": {
                "type": "timeseries",
                "requests": [
                    {
                  "q": "hello.world{*}.rollup(sum,100)"
                    }
                ],
                "title": "Hello World Rollup Function"
            }
		}
    ],
    "layout_type": "ordered",
    "description": "Dashbord created via DD API.",
    "is_read_only": true,
    "notify_list": [
        "test@datadoghq.com"
    ],
    "template_variables": [
        {
            "name": "host1",
            "prefix": "host",
            "default": "myhost"
        }
    ]
}
```

## Once this is created, access the Dashboard from your Dashboard List in the UI:
- Set the Timeboard's timeframe to the past 5 minutes
- Take a snapshot of this graph and use the @ notation to send it to yourself.
- Bonus Question: What is the Anomaly graph displaying?

### Accessing the newly created dashboard from the UI:

[Screenshot: Dashboard created](https://drive.google.com/open?id=1FlSVLHo-pRBpQzP_tnZc3LW1EgMoG8EI)

### Snapshot of my graph send to myself:

[Screenshot: Snapshot of my graph](https://drive.google.com/open?id=1MrpxaZQ20khSIfUHFG6w0vWphN7oGbrw)

### Email received:

[Screenshot: Email received](https://drive.google.com/open?id=1bVpzh5uzsA38vATeaZmAgfcZqhAjBi7Q)

### What is the Anomaly graph displaying?

The Anomaly graph uses historical data to highlight when a measured metric is outside of the expected range. Hence it is very useful when metrics have predictable patterns.

# Monitoring Data

## Create a new Metric Monitor that watches the average of your custom metric (my_metric) and will alert if it’s above the following values over the past 5 minutes:

Attributes:
- Warning threshold of 500
- Alerting threshold of 800
- And also ensure that it will notify you if there is No Data for this query over the past 10m.

[New Monitor 1/3](https://drive.google.com/open?id=1HRR4Ps5QBb_3DlPyrbJiAB3k5kY3gtve)

[New Monitor 2/3](https://drive.google.com/open?id=1UkMlIlq9PRgk0Yu15Y5qYd0JuXSPgYHe)

[New Monitor 3/3](https://drive.google.com/open?id=1pRzLGygrkMeW1ASfttuQgm7m1WIQL8q6)

Testing the monitor:

[Screenshot: Test set up](https://drive.google.com/open?id=10x1DifH7GSY688c8Q4SfoQvt6uUuz3d0)

[Screenshot: Emails received](https://drive.google.com/open?id=1VfCZJ3J0mQDlg3H57lrAFb8bHI8NifuI)

## Since this monitor is going to alert pretty often, you don’t want to be alerted when you are out of the office. Set up two scheduled downtimes for this monitor:
        One that silences it from 7pm to 9am daily on M-F,
        And one that silences it all day on Sat-Sun.
        Make sure that your email is notified when you schedule the downtime and take a screenshot of that notification.

DD Documentation:
  - [Schedule downtime](https://docs.datadoghq.com/monitors/downtimes/#schedule-downtime)

### One that silences it from 7pm to 9am daily on M-F

[Screenshot: Schedule Downtime from 7pm to 9am daily on M-F 1/2](https://drive.google.com/open?id=1YXrEk7kTrzqTYZaqXPjU3kDC0FnUjDwB)

[Screenshot: Schedule Downtime from 7pm to 9am daily on M-F 2/2](https://drive.google.com/open?id=1mNyCXsO3sEwQypvRKZTwAeTQw8oxrlzu)

### One that silences it all day on Sat-Sun

[Screenshot: Schedule Downtime Sat-Sun](https://drive.google.com/open?id=1pZU6YFer2Qp3SHPo4__Y-ZaxiV3a4mfL)

###

[Screenshot: notification](https://drive.google.com/open?id=1y0h5LcGFSygPHIiahk2yHygRMky6enE0)

# Collecting APM Data

DD Documentation
  - [Quickstart Guide - APM](https://docs.datadoghq.com/getting_started/tracing/#overview)

For this execise I created a Vagrant Ubuntu virtual machine and installed the DD agent.

To enable APM I configured the datadog.yaml:

```
...
apm_config:
  enabled: true
  env: hello_world
...
```

This is the dummy application from the docs that I will use for this exercise:

File: hello.py
```
from flask import Flask
import logging
import sys

# Have flask use stdout as the logger
main_logger = logging.getLogger()
main_logger.setLevel(logging.DEBUG)
c = logging.StreamHandler(sys.stdout)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
c.setFormatter(formatter)
main_logger.addHandler(c)

app = Flask(__name__)

@app.route('/')
def api_entry():
    return 'Entrypoint to the Application'

@app.route('/api/apm')
def apm_endpoint():
    return 'Getting APM Started'

@app.route('/api/trace')
def trace_endpoint():
    return 'Posting Traces'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port='5050')
```

Instrumenting the application in DD:

`$ ddtrace-run python hello.py`

Creating some dummy calls against the new app to generate tracing data:

`while sleep 1; do curl http://0.0.0.0:5050/; done`

My first application trace in DD:

[Screenshot: Dashboard](https://drive.google.com/open?id=1iP7ID2JyltvqVndzXSih5D1_ShTNcBJ-)

Q: What is the difference between a Service and a Resource?

A: Services are building blocks of a microservice architecture. A resource is a particular action for a given service, e.g. a web endpoint.

# Final Question

Is there anything creative you would use DataDog for?

Obviously there are many ways the DD solution can be used and the time to value should be incredibly short. On a fun side, I would like to monitor the metrics from my garmin fenix fitness watch and send alerts when I exercise too little or if there are any anomalies in my health stats.

I am interested in stock markets and investing. Maybe one could also monitor Mr. Trumps twitter account for new tariff announcements. There seems to be an undeniable correlation between his tariff announcements and a stock market decline. So an alert could be used to open a short position ;-).

In my opinion the DD solution worked great. Since I like to invest into the stock market, the technical exercise gave me a lot of insight in how the solution works and I believe it has great potential in the market. I decided to go long on DD :-). At 35,90$ per share it should work out well for me.
[Long Position on DD](https://drive.google.com/open?id=1vgI5CxDvs9SmZ0HTBKmZ-havb_gjxYaD)

[Cloud Monitoring](https://i.pinimg.com/originals/02/12/a7/0212a79757a9e3914c40d4826a87d7de.jpg)
