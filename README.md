# SWAT-5350
This repo is to track the changes and the work done to debug and fix SWAT-5350

## Problem Statement:
Customer reported that the routing processor is not routing metrics to the exporters correctly. The metrics only end up with the exporter that is first in the list.<br /><br />
Below is the routing processor config, when metrics with `metrics.platform` attributes is set to `both`, the metrics are being emitted to the splunk enterprise but not to signalfx. 

```  routing/metrics/ctc:
    attribute_source: resource
    default_exporters:
    - splunk_hec/enterprise_metrics/ctc
    drop_resource_routing_attribute: true
    from_attribute: metrics.platform
    table:
    - exporters:
      - signalfx
      expression: delete_matching_keys(resource.attributes, "com\\.splunk\\.index|metrics\\.platform|com\\.splunk\\.hec\\.     access_token")
        where IsMatch(resource.attributes["metrics.platform"], "observability") ==
        true
    - exporters:
      - splunk_hec/enterprise_metrics/ctc
      value: enterprise
    - exporters:
      - splunk_hec/enterprise_metrics/ctc
      - signalfx
      value: both
```

## Repro:
I can clearly see the issue in the logs provided, but I was not able to reproduce the problem on OTEL Splunk distro v0.62.0 <br />
The [repro](repro/) is a docker compose that starts 3 services, `Splunk enterprise`, `http server` that provides prometheus metrics and `otel v0.62.0` <br /> <br />

### Run Repro:
- Edit `otel-collector-config.yml` and replace `<realm>` and `<token>` with the actual values in the signalfx exporter
- Run `docker compose up -d`
- Wait 2 minutes for splunk service to starts and login to [http://localhost:18000](http://localhost:18000) with `admin` and password `changeme`
- Once logged in, visit the [analytics workspace](http://localhost:18000/en-US/app/search/analytics_workspace) to see the metrics collected by Splunk.
- Also check for `hz_queue_ownedItemCount` and `foo_bar` on you signalfx org