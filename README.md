# Loggui

Loggui aim to centralize all the logs from yours applications at one point and expose an web ui that display your logs as raw.

No need to deploy a huge ELK stack to see all your logs at one point.

# Installation
## Docker

```
# Launch web ui
docker run --rm -p 80:80 jbourdale/168h-app

# Launch logstash
docker run --rm -e LOGGUI_HOST="127.0.0.1" -e LOGGUI_PORT="80" jbourdale/168h-logstash
```

## Docker compose

```
docker-compose up
```

## Helm (for Kubernetes)

```
# Add the repository
helm repo add 168h https://jbourdale.github.io/168h

# Install app
helm install jbourdale/168h
```

## Baremetal

### Install the app

```
# Install dependencies
npm install

# Build the project
npm run build

# Launch
node dist/
```

### Install logstash

```

```

# Let's see our logs!

Hit `http://<YOUR_WEBUI_HOST>/` in your browser to access the web ui


# Configuration

All your logs need to be redirected to our logstash. Follow examples to configure your logging system

## Syslog

In your /etc/rsyslog.conf just add the following line

```
*.* @<YOUR_LOGSTASH_HOST>:1514
```

## Fluentd

See https://docs.fluentd.org/configuration/config-file to find your configuration file

In this file, add an output as follow:

```
    <match **>
        @type forward
        time_as_integer true
        <server>
            host <YOUR_LOGSTASH_HOST>
            port 24114
        </server>
    </match>
```

## Filebeat

Your configuration file will be locatated by default in `/etc/filebeat/filebeat.yml`

Add the following lines to this file
```
#----------------------------- Logstash output --------------------------------
output.logstash:
  hosts: ["<YOUR_LOGSTASH_HOST>:5044"]
```

## Fluentbit

In your fluentbit configuration file, add an output like the following one :

```
[OUTPUT]
    Name   http
    Match  *
    Host   <YOUR_LOGSTASH_HOST>
    Port   6349
    Format json
```

## Python, Django and Gunicorn

We strongly recommand the `python3-logstash` package that you can find at https://github.com/israel-fl/python3-logstash

Use the port `7001`.

### Python

```
test_logger = logging.getLogger('python-logstash-logger')
test_logger.setLevel(logging.INFO)
test_logger.addHandler(logstash.LogstashHandler(<YOUR_LOGSTASH_HOST>, 7001, version=1))
```

### Django
In your `settings.py`
```
LOGGING = {
    ...
    'handlers': {
        'logstash': {
            'level': 'DEBUG',
            'class': 'logstash.LogstashHandler',
            'host': '<YOUR_LOGSTASH_HOST>',
            'port': 7001,
            'version': 1, # Version of logstash event schema. Default value: 0 (for backward compatibility of the library)
            'message_type': 'logstash',  # 'type' field in logstash message. Default value: 'logstash'.
            'fqdn': False, # Fully qualified domain name. Default value: false.
            'tags': ['tag1', 'tag2'], # list of tags. Default: None.
        },
    },
    ...
}
```

### Gunicorn

```
[handler_logstash]
class=logstash.TCPLogstashHandler
formatter=json
args=('<YOUR_LOGSTASH_HOST>',7001)
```

## Node

We strongly recommand the npm package `winston-logstash` https://www.npmjs.com/package/winston-logstash

Use it as follow with port 7001:

```
  var winston = require('winston');

  //
  // Requiring `winston-logstash` will expose
  // `winston.transports.Logstash`
  //
  require('winston-logstash');

  winston.add(winston.transports.Logstash, {
    port: 7001,
    node_name: 'my node name',
    host: '<YOUR_LOGSTASH_HOST>'
  });
```

## Java

Using Log4j to send logs to logstash is deprecated (https://www.elastic.co/guide/en/logstash/current/plugins-inputs-log4j.html).

Their strongly recommand to use filebeat instead.
