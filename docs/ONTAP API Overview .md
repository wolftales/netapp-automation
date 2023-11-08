# ONTAP 9.8 REST API Overview and Account role setup

## Creating an account that can access REST APIs but won't allow access to System Manager.

Some caveats:
* ONTAP has builtin roles - both ontapi & rest roles
* When `-role readonly` && `-application http` are provided login has:
  * access to RO REST && RO System Manager
    * Ror a limited audience this could be fine - admin normal accounts, etc.
    * For end-users, this may not be desireable
* Restricting access to only REST && Readonly requires a custom role, see below  

### Steps to create a custom role for REST only access  

1. Create a new role leveraging the `security login rest-role create`
  * Considerations:
    * should this be at the SVM layer and not the cluster?
  * Tested config:
    * `security login rest-role create -vserver testTap -role api_ro -api /api -access readonly`

2. Create a user (add group) login that leverages this role
  * Tested config:
    * rest-user
    * application: http
    * auth_method: Password
    * role: api_ro

3. Test the REST API sends & System Manager login
   1. REST RO calls should work
   2. System Manager access should not.  

The above should enable API access to the user (or group) that is configured to send REST calls.  

## Target APIs

Notes: All volumes are flexgroups

| Performance Metric          | API                     | Status  - Example #   |
|-----------------------------|-------------------------|-------------|
| Node CPU Utilization        | N/A                     | No          |
| Node Through per interface  | N/A                     | No          |
| Vol Capacity total & Used   | /storage/volumes        | Yes - 4     |
| Vol Snapshot Used           | /storage/volumes        | Yes - 5     |
| Vol Perf: IOPs & Throughput  | /storage/volumes       | Yes - 6     |


## The API methods

```
API endpoint: https://<ONTAP_IP_ADDRESS>/api
API Documentation: https://<ONTAP_IP_ADDRESS>/docs/api
```

`curl` example  

`curl -X GET -u <rest-user>:<PASSWORD> -k 'https://<ONTAP_IP_ADDRESS>/api/storage/volumes'`

For volume API calls, the process will normally be:

## 1. Get volume of interest UUID:

Command:
``` 
curl --location --request GET 'https://192.168.7.187/api/storage/volumes?name=fg_01' \
--header 'Authorization: Basic cmVzdC11c2VyOm5ldGFwcDEyMzQ=' 
```

Output:
```
{
    "records": [
        {
            "uuid": "c49ce512-3f5b-11ed-9c79-005056b3e9e3",
            "name": "fg_01",
            "_links": {
                "self": {
                    "href": "/api/storage/volumes/c49ce512-3f5b-11ed-9c79-005056b3e9e3"
                }
            }
        }
    ],
    "num_records": 1,
    "_links": {
        "self": {
            "href": "/api/storage/volumes?name=fg_01"
        }
    }
}
```


## 2. Display all the information for a specific volume

Command:
```
curl --location --request GET 'https://192.168.7.187/api/storage/volumes/c49ce512-3f5b-11ed-9c79-005056b3e9e3' \
--header 'Authorization: Basic cmVzdC11c2VyOm5ldGFwcDEyMzQ='
```

Ouput:
```
{
    "uuid": "c49ce512-3f5b-11ed-9c79-005056b3e9e3",
    "comment": "",
    "create_time": "2022-09-28T11:31:29-07:00",
    "language": "c.utf_8",
    "name": "fg_01",
    "size": 1099511627776,
    "state": "online",
    "style": "flexgroup",
    "tiering": {
        "policy": "none"
    },
    "cloud_retrieval_policy": "default",
    "type": "rw",
    "aggregates": [
        {
            "name": "testTap_01_aggr1",
            "uuid": "daa21ebc-3e72-11ed-9c79-005056b3e9e3"
        }
    ],
    "clone": {
        "is_flexclone": false
    },
    "nas": {
        "export_policy": {
            "name": "default"
        }
    },
    "snapshot_policy": {
        "name": "default"
    },
    "svm": {
        "name": "vs01",
        "uuid": "14e7233d-3e73-11ed-9c79-005056b3e9e3",
        "_links": {
            "self": {
                "href": "/api/svm/svms/14e7233d-3e73-11ed-9c79-005056b3e9e3"
            }
        }
    },
    "space": {
        "size": 1099511627776,
        "available": 80260583424,
        "used": 1747865600
    },
    "snapmirror": {
        "is_protected": false
    },
    "metric": {
        "timestamp": "2022-09-28T18:50:45Z",
        "duration": "PT15S",
        "status": "ok",
        "latency": {
            "other": 0,
            "total": 48,
            "read": 37,
            "write": 59
        },
        "iops": {
            "read": 3672,
            "write": 3671,
            "other": 0,
            "total": 7343
        },
        "throughput": {
            "read": 15042696,
            "write": 15037508,
            "other": 0,
            "total": 30080204
        },
        "cloud": {
            "timestamp": "2022-09-28T18:50:45Z",
            "status": "ok",
            "duration": "PT15S",
            "iops": {
                "read": 0,
                "write": 0,
                "other": 0,
                "total": 0
            },
            "latency": {
                "read": 0,
                "write": 0,
                "other": 0,
                "total": 0
            }
        }
    },
    "analytics": {
        "state": "off"
    },
    "_links": {
        "self": {
            "href": "/api/storage/volumes/c49ce512-3f5b-11ed-9c79-005056b3e9e3"
        }
    }
}
```


## 3. Example for selecting specific fields:

Command:
```
curl --location --request GET 'https://192.168.7.187/api/storage/volumes/c49ce512-3f5b-11ed-9c79-005056b3e9e3?fields=metric.latency,metric.throughput,space' \
--header 'Authorization: Basic cmVzdC11c2VyOm5ldGFwcDEyMzQ='
```

Output:
```
{
    "uuid": "c49ce512-3f5b-11ed-9c79-005056b3e9e3",
    "name": "fg_01",
    "space": {
        "size": 1099511627776,
        "available": 80260583424,
        "used": 1754431488,
        "over_provisioned": 1017496612864,
        "snapshot": {
            "used": 0,
            "reserve_percent": 5,
            "autodelete_enabled": false
        }
    },
    "metric": {
        "latency": {
            "other": 0,
            "total": 46,
            "read": 35,
            "write": 58
        },
        "throughput": {
            "read": 15559338,
            "write": 15532578,
            "other": 0,
            "total": 31091916
        }
    },
    "_links": {
        "self": {
            "href": "/api/storage/volumes/c49ce512-3f5b-11ed-9c79-005056b3e9e3"
        }
    }
}
```


## 4. Volume Capacity Total & Used:

Command:
```
curl --location --request GET 'https://192.168.7.187/api/storage/volumes/c49ce512-3f5b-11ed-9c79-005056b3e9e3?fields=space.size,space.used' \
--header 'Authorization: Basic cmVzdC11c2VyOm5ldGFwcDEyMzQ='
```

Output:
```
{
    "uuid": "c49ce512-3f5b-11ed-9c79-005056b3e9e3",
    "name": "fg_01",
    "space": {
        "size": 1099511627776,
        "used": 1760919552
    },
    "_links": {
        "self": {
            "href": "/api/storage/volumes/c49ce512-3f5b-11ed-9c79-005056b3e9e3"
        }
    }
}
```


## 5. Volume Snapshot Capacity Used:

Command:
```
curl --location --request GET 'https://192.168.7.187/api/storage/volumes/c49ce512-3f5b-11ed-9c79-005056b3e9e3?fields=space.snapshot' \
--header 'Authorization: Basic cmVzdC11c2VyOm5ldGFwcDEyMzQ='
```

Output:
```
{
    "uuid": "c49ce512-3f5b-11ed-9c79-005056b3e9e3",
    "name": "fg_01",
    "space": {
        "snapshot": {
            "used": 2042376192,
            "reserve_percent": 5,
            "autodelete_enabled": false
        }
    },
    "_links": {
        "self": {
            "href": "/api/storage/volumes/c49ce512-3f5b-11ed-9c79-005056b3e9e3"
        }
    }
}
```



## 6. Volume Performance IOPs & Throughput:

Command:
```
curl --location --request GET 'https://192.168.7.187/api/storage/volumes/c49ce512-3f5b-11ed-9c79-005056b3e9e3?fields=metric.iops,metric.throughput' \
--header 'Authorization: Basic cmVzdC11c2VyOm5ldGFwcDEyMzQ='
```

Output:
```
{
    "uuid": "c49ce512-3f5b-11ed-9c79-005056b3e9e3",
    "name": "fg_01",
    "metric": {
        "iops": {
            "read": 2226,
            "write": 2231,
            "other": 0,
            "total": 4458
        },
        "throughput": {
            "read": 72948121,
            "write": 73131622,
            "other": 0,
            "total": 146079744
        }
    },
    "_links": {
        "self": {
            "href": "/api/storage/volumes/c49ce512-3f5b-11ed-9c79-005056b3e9e3"
        }
    }
}
```





Added in ONTAP 9.11.1 - Performance Counters (For Future Reference)

In later releases of ONTAP the performance metrics get more comprehensive.

```
https://docs.netapp.com/us-en/ontap-automation/migrate/performance-counters.html#discover-the-available-performance-counter-tables

curl -X GET -u admin:<PASSWORD> -k 'https://<ONTAP_IP_ADDRESS>/api/cluster/counter/tables'

        "name": "processor",
        "_links": {
          "self": {
            "href": "/api/cluster/counter/tables/processor"
          }
        }
      },
      {
        "name": "processor:node",
        "_links": {
          "self": {
            "href": "/api/cluster/counter/tables/processor%3Anode"

Example
curl -X GET -u admin:<PASSWORD> -k 'https://<ONTAP_IP_ADDRESS>/api/cluster/counter/tables/host_adapter'

curl -X GET -u admin:<PASSWORD> -k 'https://<sandbox>/api/cluster/counter/tables/processor'

curl -X GET -u admin:<PASSWORD> -k 'https://<sandbox>/api/cluster/counter/tables/processor/rows'

curl -X GET -u admin:<PASSWORD> -k 'https://<sandbox>/api/cluster/counter/tables/processor/rows/sandbox-01:processor0'
```