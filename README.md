# selenosis
Scalable, stateless selenium hub for Kubernetes cluster.

## Overview
### Available flags
```
[user@host]# ./selenosis --help
Scallable, stateless selenium grid for Kubernetes cluster

Usage:
  selenosis [flags]

Flags:
      --port string                          port for selenosis (default ":4444")
      --proxy-port string                    proxy continer port (default "4445")
      --browsers-config string               browsers config (default "config/browsers.yaml")
      --namespace string                     kubernetes namespace (default "default")
      --service-name string                  kubernetes service name for browsers (default "selenosis")
      --browser-wait-timeout duration        time in seconds that a browser will be ready (default 30s)
      --session-wait-timeout duration        time in seconds that a session will be ready (default 1m0s)
      --session-iddle-timeout duration       time in seconds that a session will iddle (default 5m0s)
      --session-retry-count int              session retry count (default 3)
      --graceful-shutdown-timeout duration   time in seconds  gracefull shutdown timeout (default 5m0s)
  -h, --help                                 help for selenosis

```

### Available endpoints
| Protocol | Endpoint                    |
|--------- |---------------------------- |
| HTTP    | /wd/hub/session              |
| HTTP    | /wd/hub/session/{sessionId}/ |
| HTTP    | /wd/hub/status               |
| WS      | /vnc/{sessionId}             |
| WS/HTTP | /devtools/{sessionId}        |
| HTTP    | /download/{sessionId}        |
| HTTP    | /clipboard/{sessionId}       |
<br/>

## Configuration
Selenosis requires config to start browsers in K8 cluster. Config can be JSON or YAML file.<br/>
Basic configuration be like (all fields in this example are mandatory):

```json
{
    "chrome": {
        "path": "/",
        "versions": {
            "85.0": {
                "image": "selenoid/vnc:chrome:85.0"
            },
            "86.0": {
                "image": "selenoid/vnc:chrome:86.0"
            }
        }
    },
    "firefox": {
        "path": "/wd/hub",
        "versions": {
            "81.0": {
                "image": "selenoid/vnc:firefox_81.0"
            },
            "82.0": {
                "image": "selenoid/vnc:firefox_82.0"
            }
        }
    },

    "opera" : {
        "path": "/",
        "versions": {
            "70.0": {
                "image": "selenoid/vnc:opera_70.0"
            },
            "71.0": {
                "image": "selenoid/vnc:opera_71.0"
            }
        }
    }
}
```
``` yaml
---
chrome:
  path: "/"
  versions:
    '85.0':
      image: selenoid/vnc:chrome:85.0
    '86.0':
      image: selenoid/vnc:chrome:86.0
firefox:
  path: "/wd/hub"
  versions:
    '81.0':
      image: selenoid/vnc:firefox_81.0
    '82.0':
      image: selenoid/vnc:firefox_82.0
opera:
  path: "/"
  versions:
    '70.0':
      image: selenoid/vnc:opera_70.0
    '71.0':
      image: selenoid/vnc:opera_71.0
```

Browser name and browser version are taken from Selenium desired capabilities.<br/>

Each browser can have default <b>spec/annotations/labels</b>, they will merged to all browsers listed in the <b>versions</b> section.

``` json
{
  "chrome": {
    "path": "/",
    "meta": {
      "labels": {
        "environment": "aqa",
        "app": "myCoolApp"
      },
      "annotations": {
        "build": "dev-v1.11.2",
        "builder": "jenkins"
      }
    },
    "spec": {
      "resources": {
        "requests": {
          "memory": "500Mi",
          "cpu": "0.5"
        },
        "limits": {
          "memory": "1000Gi",
          "cpu": "1"
        }
      },
      "hostAliases": [
        {
          "ip": "127.0.0.1",
          "hostnames": [
            "foo.local",
            "bar.local"
          ]
        },
        {
          "ip": "10.1.2.3",
          "hostnames": [
            "foo.remote",
            "bar.remote"
          ]
        }
      ],
      "env": [
        {
          "name": "TZ",
          "value": "Europe/Kiev"
        },
        {
          "name": "SCREEN_RESOLUTION",
          "value": "1920x1080x24"
        },
        {
          "name": "ENABLE_VNC",
          "value": "true"
        }
      ]
    },
    "versions": {
      "85.0": {
        "image": "selenoid/vnc:chrome:85.0"
      },
      "86.0": {
        "image": "selenoid/vnc:chrome:86.0"
      }
    }
  }
}
```

``` yaml
---
chrome:
  path: "/"
  meta:
    labels:
      environment: aqa
      app: myCoolApp
    annotations:
      build: dev-v1.11.2
      builder: jenkins
  spec:
    resources:
      requests:
        memory: 500Mi
        cpu: '0.5'
      limits:
        memory: 1000Gi
        cpu: '1'
    hostAliases:
    - ip: 127.0.0.1
      hostnames:
      - foo.local
      - bar.local
    - ip: 10.1.2.3
      hostnames:
      - foo.remote
      - bar.remote
    env:
    - name: TZ
      value: Europe/Kiev
    - name: SCREEN_RESOLUTION
      value: 1920x1080x24
    - name: ENABLE_VNC
      value: 'true'
  versions:
    '85.0':
      image: selenoid/vnc:chrome:85.0
    '86.0':
      image: selenoid/vnc:chrome:86.0
```
You can override default browser <b>spec/annotation/labels</b> by providing individual <b>spec/annotation/labels</b> to browser version
``` json
{
  "chrome": {
    "path": "/",
    "meta": {
      "labels": {
        "environment": "aqa",
        "app": "myCoolApp"
      },
      "annotations": {
        "build": "dev-v1.11.2",
        "builder": "jenkins"
      }
    },
    "spec": {
      "resources": {
        "requests": {
          "memory": "500Mi",
          "cpu": "0.5"
        },
        "limits": {
          "memory": "1000Gi",
          "cpu": "1"
        }
      },
      "hostAliases": [
        {
          "ip": "127.0.0.1",
          "hostnames": [
            "foo.local",
            "bar.local"
          ]
        },
        {
          "ip": "10.1.2.3",
          "hostnames": [
            "foo.remote",
            "bar.remote"
          ]
        }
      ],
      "env": [
        {
          "name": "TZ",
          "value": "Europe/Kiev"
        },
        {
          "name": "SCREEN_RESOLUTION",
          "value": "1920x1080x24"
        },
        {
          "name": "ENABLE_VNC",
          "value": "true"
        }
      ]
    },
    "versions": {
      "85.0": {
        "image": "selenoid/vnc:chrome:85.0",
        "spec": {
          "resources": {
            "requests": {
              "memory": "750Mi",
              "cpu": "0.5"
            },
            "limits": {
              "memory": "1500Gi",
              "cpu": "1"
            }
          }
        }
      },
      "86.0": {
        "image": "selenoid/vnc:chrome:86.0",
        "spec": {
          "hostAliases": [
            {
              "ip": "127.0.0.1",
              "hostnames": [
                "bla-bla.com"
              ]
            }
          ]
        },
        "meta": {
          "labels": {
            "environment": "dev",
            "app": "veryCoolApp"
          }
        }
      }
    }
  }
}
```
``` yaml
---
chrome:
  path: "/"
  meta:
    labels:
      environment: aqa
      app: myCoolApp
    annotations:
      build: dev-v1.11.2
      builder: jenkins
  spec:
    resources:
      requests:
        memory: 500Mi
        cpu: '0.5'
      limits:
        memory: 1000Gi
        cpu: '1'
    hostAliases:
    - ip: 127.0.0.1
      hostnames:
      - foo.local
      - bar.local
    - ip: 10.1.2.3
      hostnames:
      - foo.remote
      - bar.remote
    env:
    - name: TZ
      value: Europe/Kiev
    - name: SCREEN_RESOLUTION
      value: 1920x1080x24
    - name: ENABLE_VNC
      value: 'true'
  versions:
    '85.0':
      image: selenoid/vnc:chrome:85.0
      spec:
        resources:
          requests:
            memory: 750Mi
            cpu: '0.5'
          limits:
            memory: 1500Gi
            cpu: '1'
    '86.0':
      image: selenoid/vnc:chrome:86.0
      spec:
        hostAliases:
        - ip: 127.0.0.1
          hostnames:
          - bla-bla.com
      meta:
        labels:
          environment: dev
          app: veryCoolApp

```
## Deployment
Files and steps required for selenosis deployment available in [selenosis-deploy](https://github.com/alcounit/selenosis-deploy) repository

 ## Run yout tests
 ``` java
 DesiredCapabilities capabilities = new DesiredCapabilities();
capabilities.setBrowserName("chrome");
capabilities.setVersion("85.0");
capabilities.setCapability("enableVNC", true);
capabilities.setCapability("enableVideo", false);

RemoteWebDriver driver = new RemoteWebDriver(
    URI.create("http://<loadBalancerIP|nodeIP>:<port>/wd/hub").toURL(), 
    capabilities
);
 ```
  ``` python
from selenium import webdriver
        
capabilities = {
    "browserName": "chrome",
    "version": "85.0",
    "enableVNC": True,
    "enableVideo": False
}

driver = webdriver.Remote(
    command_executor="http://<loadBalancerIP|nodeIP>:<port>/wd/hub",
    desired_capabilities=capabilities)
 ```

## Features
### Scalability
By default selenosis starts with 2 replica sets. To change it, edit selenosis deployment file: <b>03-selenosis.yaml</b>
``` yaml

apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: selenosis
  namespace: selenosis
spec:
  replicas: 2
  selector:
...
```

### Stateless
Selenosis doesn't store any session info. All connections to the browsers are automatically assigned via headless service.
