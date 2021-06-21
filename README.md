# Using Apigee Message Logger with fluent bit

Use the [Message Logger](https://docs.apigee.com/api-platform/reference/policies/message-logging-policy) policy to log messages to [Fluent Bit](https://docs.fluentbit.io/manual/about/what-is-fluent-bit).

## Scenario

Apigee's Message Logger policy allows users to forward log messages (parts or whole of the request and/or response) to a remote syslog server (or the file system in Apigee Private Cloud). Enterprises may have Splunk or Dynatrace as their logging standard and want to integrate Apigee Message Logger with such standards.

Fluent Bit is an open source data collector for unified logging layer and has a minimal set of [input plugins](https://docs.fluentbit.io/manual/concepts/data-pipeline/input) that allows enterprises to integrate with. This example will show how to setup Fluent Bit to consume messages from Message Logger.

### Fluent Bit setup

Here is the sample configuration used for [Fluent Bit](./fluent-bit-logger-tcp.yaml). The configuration for Fluent Bit is stored in a ConfigMap with the following details:

```
    # Takes the messages sent over TCP
    [INPUT]
        Name         syslog
        Listen       0.0.0.0
        Port         5140
        Mode         tcp
        Parser       syslog-rfc5424
```

This configuration listens on TCP on port 5140. Install the Fluent Bit service with the command:

```bash
kubectl apply -f fluent-bit-logger-tcp.yaml
```

### Message Logger Policy

Configure the Message Logger policy to send syslog events to the Fluent Bit service. A sample API proxy is included [here](./apiproxy)

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<MessageLogging async="false" continueOnError="false" enabled="true" name="Log-Response">
    <DisplayName>Log Response</DisplayName>
    <Syslog>
        <Message>[tag="{organization.name}.{apiproxy.name}.{environment.name}"] {response.content}</Message>
        <Host>apigee-fluent-bit-tcp.apps.svc.cluster.local</Host>
        <Port>5140</Port>
        <Protocol>TCP</Protocol>
        <FormatMessage>true</FormatMessage>
    </Syslog>
</MessageLogging>
```

### Output

If the setup was successful, you will see logs in Fluent Bit's pod

Access the pod's log

```bash
kubectl logs -n apps apigee-fluent-bit-tcp-7d8c5dcdb5-5695h
```

OUTPUT:

```sh
Fluent Bit v1.7.8
* Copyright (C) 2019-2021 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io
[2021/06/17 16:59:03] [ info[] [engine[] started (pid=1)
[2021/06/17 16:59:03] [ info[] [storage[] version=1.1.1, initializing...
[2021/06/17 16:59:03] [ info[] [storage[] in-memory
[2021/06/17 16:59:03] [ info[] [storage[] normal synchronization mode, checksum disabled, max_chunks_up=128
[2021/06/17 16:59:03] [ info[] [in_syslog[] TCP server binding 0.0.0.0:5140
[2021/06/17 16:59:03] [ info[] [sp[] stream processor started
[0[] syslog.0: [1624285503.297000000, {"pri"=>"14", "time"=>"2021-06-21T14:25:03.297+0000", "host"=>"1a49a012-b041-43f6-9dca-365ccd50bba9", "ident"=>"Apigee-Edge", "pid"=>"-", "msgid"=>"-", "extradata"=>"[tag="srinandans-hybrid.mocktarget.prod2"]", "message"=>"{"firstName":"John","lastName":"Doe","city":"San Jose","state":"CA"}"}]
```

___

## Support

This is not an officially supported Google product
