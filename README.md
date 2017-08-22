# Prometheus AlertManager
[Prometheus](https://prometheus.io/) is a powerful, open-source monitoring solution. This integration to xMatters extends the alerting capabilities of AlertManager to notify the right people at the right time. 


# Pre-Requisites
* Prometheus with [AlertManager](https://github.com/prometheus/alertmanager) set up and running. 
* An application to monitor
* xMatters account - If you don't have one, [get one](https://www.xmatters.com)!

# Files
* [Prometheus.zip](Prometheus.zip) - Comm Plan for the integration builder script and notification form templates. 

# How it works
[Alert rules](https://prometheus.io/docs/alerting/rules/) are defined in Prometheus and sent to AlertManager for further processing. The AlertManager [config file](https://prometheus.io/docs/alerting/configuration/#configuration-file) defines what happens after the alerts are sent to AlertManager. Using the xMatters notifier, an inbound integration endpoint can be defined and tied to a receiver, which can then be referenced by a route. Once the alert reaches xMatters, the integration builder script transforms the content and builds the event, sets the recipient to the receiver and creates the event. 

# Installation

## xMatters set up
1. Import the [Prometheus.zip](Prometheus.zip) comm plan. 
2. Click Edit > Integration Builder and expand the Inbound Integrations section. 
3. Click on the `Inbound from Prometheus` link and copy the endpoint at the bottom of the page. Save for later. 


## Prometheus set up
1. Open the `alertmanager.yml` file and navigate to the `receivers` section. The location of the file and the section will depend on the details of the installation. 
2. Add a new receiver. The name of the receiver will be the recipients of the event. An optional `priority` field can be passed to indicate `high`, `medium`, or `low` priority. For example, to target the `Database` group:

```yaml
- name: 'Database'
  xmatters_configs:
    - url: 'https://acme.xmatters.com/api/integration/1/functions/UUID/triggers?apiKey=KEY'
      priority: 'high'
```

3. Edit the route that should target the new receiver. For example, to notifiy this `Database` receiver for the `octoapp` service:

```yaml
  routes:
  - match_re:
      service: ^(octoapp)$
    receiver: Database
```

4. Repeat as needed for new routes and new receivers. 

# Testing
Create or edit an Alert Rule in the alert rules file (defined in the `prometheus.yml` file) that is easy to fire. For example, to fire when the `widget_gauge` is greater than 30 for 1 minute:

```
ALERT octo_alert
  IF widget_gauge > 30
  FOR 1m
  LABELS { 
    service  = "octoapp"
  }
  ANNOTATIONS {
     summary = "The summary goes here",
     description = "The description goes here"
  }
```

Then in the monitored application, get the `widget_gauge` value above 30 for 1 minute. This will trigger an alert in AlertManager, and then will be fired off to xMatters. Make sure you have a `Database` group with a user. 

A notification will be sent out targeting the Database group:

<kbd>
	<img src="media/notification.png">
</kbd>



# Troubleshooting
Check the AlertManager log (probably in `/var/log/prometheus`, but will depend on the installation details) for any errors making the call to xMatters. Then check the Activity Stream in the `Inbound from Prometheus` script for errors. 

