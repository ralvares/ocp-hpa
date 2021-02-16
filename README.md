# ocp-hpa

Instructions to demo HPA on OCP

deploy pod which serves PHP webpage with expensive computations

    oc new-project demohpa
    git clone https://github.com/ralvares/ocp-hpa
    oc new-app . --name=php-apache
    oc set resources deployment/php-apache --limits=cpu=500m --requests=cpu=200m

Create horizontal pod autoscaler to scale pod based on CPU load-

    oc autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10

Create pod with shell we will use to request that PHP webpage

    oc run -it --rm load-generator --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache:8080; done"

Hit enter to reach command prompt

You can view CPU metrics by watching the HPA resource, and see new pods being spun up by watching the RS

    watch -n 0.5 oc get hpa
    watch -n 0.5 oc get rs

Important notes about HPAs-
- Requires resource requests and limits at the pod level
- Will only create additional pods. This can produce problems if nodes start to become tightly scheduled or run out of space. Solutions:
    - ClusterAutoScaler to provision additional nodes
    - Use pod priorities to ensure that critical pods are always able to be scheduled
