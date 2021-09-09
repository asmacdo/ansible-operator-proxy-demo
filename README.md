This repo is a demonstration of a proxy-friendly Ansible-based Operator
using a real proxy.

To use, deploy the operator and create the CR
`make deploy && kubectl apply -f config/samples/demo_v1alpha1_ansibleproxydemo.yaml`

The proxy and curl pods will be running:
```bash
kubectl get pods -n default
```

```bash
NAME        READY   STATUS    RESTARTS   AGE
curl-pod    1/1     Running   0          3m22s
proxy-pod   1/1     Running   0          3m22s
```
Note the curl-pod does nothing, it exists to `exec` into for hand
testing.

The Operator should have come up configured to use the proxy IP as
`HTTP_PROXY`, which applies to the Operator env and is passed to the
`Job` operand as well. The `env-job` runs a `curl http://example.com`
and exits.

If everything is working as expected you will see that this request was
proxied.

```bash
kubectl logs -n default proxy-pod
```
```bash
Proxy server listening at http://*:8080
10.244.0.77:54946: client connect
10.244.0.77:54946: server connect example.com:80 (93.184.216.34:80)
10.244.0.77:54946: GET http://example.com/
                << 200 OK 1.23k
10.244.0.77:54946: client disconnect
10.244.0.77:54946: server disconnect example.com:80 (93.184.216.34:80)
```
### Variable Passdown Notes
The passdown in `roles/ansibleproxydemo/taks/main.yml` specifies the
environment variables using `lookup`.

### Setup Notes

In order to set the env vars in the Operator environment, I made use of
`envsubst`. See the Makefile target `deploy` and
`config/manager/manager.yaml:37` for how this works.

