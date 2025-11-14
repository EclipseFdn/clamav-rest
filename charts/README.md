## Accessing clamav-rest installation - EF specific

clamav-rest is available as application service, inside deployments.
That means, all the pods in deployment and namespace can reach the service.


Example:

Running curl from inside a pod in the same namespace as clamav-rest:
```
curl http://clamav-rest-staging:9000/

{"Pools":"1","State":"STATE: VALID PRIMARY","Threads":"THREADS: live 1  idle 0 max 10 idle-timeout 30","Memstats":"MEMSTATS: heap N/A mmap N/A used N/A free N/A releasable N/A pools 1 pools_used 1373.773M pools_total 1373.820M","Queue":"QUEUE: 0 items"}
```

There's also route defined as https (self signed) and FQDN address works as well from inside EF infrastructure.

In order to access from committers workstations, configuration mentioned in https://gitlab.eclipse.org/eclipsefdn/it/releng/internal-services should be followed.

Please read README.md in the root of this repository for more information about accessing clamav-rest and appropriate URLs.


Tests:

```
~$ curl -ks https://clamav-rest-staging-open-vsx-org-staging.apps.okd-c1.eclipse.org/version | jq '.'
{
  "Clamav": "1.4.2",
  "Signature": "27816",
  "Signature_date": "Fri Nov  7 11:51:16 2025"
}

~$ echo 'X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*' | curl -ks  -X POST -F "file=@-" https://clamav-rest-staging-open-vsx-org-staging.apps.okd-c1.eclipse.org/v2/scan | jq '.'
[
  {
    "Status": "FOUND",
    "Description": "Eicar-Signature",
    "FileName": "-"
  }
]

~$ curl -k https://clamav-rest-staging-open-vsx-org-staging.apps.okd-c1.eclipse.org/metrics
# HELP go_build_info Build information about the main Go module.
# TYPE go_build_info gauge
go_build_info{checksum="",path="github.com/ajilach/clamav-rest",version="(devel)"} 1
# HELP go_cgo_go_to_c_calls_calls_total Count of calls made from Go to C by the current process. Sourced from /cgo/go-to-c-calls:calls.
# TYPE go_cgo_go_to_c_calls_calls_total counter
go_cgo_go_to_c_calls_calls_total 0
...
```


## Deploying to staging
Configuration for staging environment is in `values-staging.yaml` file.

To deploy or upgrade clamav-rest in staging environment, run the command below from charts/clamav-rest directory:


```
helm upgrade --install clamav-rest . -f values-staging.yaml -n open-vsx-org-staging
```

TODO:
- improve Dockerfile
- Add Jenkins job
- If accepted:
    - Add monitoring configuration - move logs to alloy
    - Implement operator in seperate repository
    - Add chart to open-vsx-org repo
    - Add production deployment configuration
