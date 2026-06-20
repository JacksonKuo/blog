---
layout: post
title: "Application QoL: Helm Deploy Flags"
date: 2026-6-20
tags: ["app"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Improve the quality of life of my app environment by adding helm deploy flags to enable or disable services.

# Helm Services
```shell
kubectl get pods
NAME                              READY   STATUS    RESTARTS       AGE
redis-985fffd47-j2cz6             1/1     Running   3 (285d ago)   293d
smokescreen-5b9d74b464-gct96      1/1     Running   1 (285d ago)   293d
springboot-app-66684d7cdd-kpvnl   1/1     Running   0              19m
```

Let's turn off smokescreen and redis. The change is pretty simple:
In the [value.yaml](https://github.com/JacksonKuo/app-springboot/blob/main/springboot-chart/values.yaml) add deploy flags:

```yaml
enableRedis: true
enableSmokescreen: true
```

And then in the `deployment-*.yaml` and `service-*.yaml` file add condition checks in the template:

{% raw %}
```yaml
{{- if .Values.enableRedis }}
...
{{- end }}
```
{% endraw %}

Apparently the endpoint logic i wrote previously doesn't cause the app to crash if the services aren't available... nice. 

* [https://bakacore.com:8443/smokescreen](https://bakacore.com:8443/smokescreen)
    * Response: `Failed to fetch data: null`
* [https://bakacore.com:8443/ratelimit](https://bakacore.com:8443/ratelimit)
    * Response: `Status=500`

But my homepage is still working [https://bakacore.com:8443/](https://bakacore.com:8443/): `Greetings random user: 96! Reload!`. Excellent. 

