# Scaling Pods on custom metrics in AKS

0. Kubernetes V 1.13

Deploy 1-4. This will create a Prom instance you can query on a pub IP. 
## Get Prom running

1. File 01 will create an deployment with service which delivers a metrics file on svc-ip:10000/metrics. This will be grapped by Prometheus. Content can be adjusted my changing the file in /myserve/index.html.
2. File 02 will create Prom CRDs.
3. File 03 crates a prom svc montior. Basically this tells an instance of Prometheus where (in which service & port) to look for metrics.
4. File 04 creates the Prometheus instance which will do what we described in 03. The prom instance picks up all svc mons with the matching label. Besides we expose Prom as a svc. 
5. Now you will be able to see your metrics in the browser on the Prom dashboard. (check kubectl get svc to get the ip and go there to port 9090)    
6. You can also check if you can query the app. (<Ip>/metrics in browser)

## Install the adaptor
7. In case you need Tiller installed, run the 05a_Tiller file and run helm init --upgrade
8. Create an adaptor to make prom info available via api
    ```
    helm install stable/prometheus-adapter --name prometheus-adapter -f 05_values.yaml    
    ```
   In the 05_values file specify:
    - the correct path to your prom svc.
    - the correct metric name.
   Defaults should work for this example.

9. This will show output with an url. 
10. Do the following query to see if everything went well:
    ```
    # query metrics from the commandline    
    kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1/namespaces/*/metrics/http_request | jq . | grep value
    Ideally you get a result like:
    ```
    "value": "20"
    ```
    
    # query available apiservices
    ```
    kubectl api-versions
    ```
    You should see custom.metrics.k8s.io now. 
    Run k describe apiservice v1beta1.custom.metrics.k8s.io

    If you get Status:True things are looknig good.
    

11. Create a sample app via deployment (eg nginx) and an hpa running k apply -f 06.
12. Now things are getting interesting. Take a look at your hpa using k get hpa.
    if you get a TARGETS entry like 1/1 things are good. If you get "<unknown>" things are gone bad. 
    ```
    k describe hpa nginx-hpa
    ```


Tried:
- god mode (didnt fix it)
- manually configure apireg (xx and xy files in skip folder)


## Things to do
2. Modify min/max in hpa to make your # of pods scale
3. Change the number in /myserver/metrics/index.html to see hpa react to changing metrics
1. Check the prom config maps to adjust your metrics







###
helpful links


- https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/
- https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/#apiservice-v1-apiregistration-k8s-io
- https://blog.jetstack.io/blog/resource-and-custom-metrics-hpa-v2/#the-custom-metrics-api
- https://medium.com/uptime-99/kubernetes-hpa-autoscaling-with-custom-and-external-metrics-da7f41ff7846
- https://github.com/lachie83/rps-demo-prometheus
- https://github.com/directxman12/k8s-prometheus-adapter