# Amalgam8 examples

Sample microservice-based applications and local sandbox environment for Amalgam8

## Table of Contents

* [Overview](#overview)
* [Amalgam8 on Kubernetes (local)](#local-k8s)
* [Amalgam8 on IBM Bluemix](#bluemix)
* [Amalgam8 on Marathon/Mesos (local)](#local-marathon)
* [Amalgam8 on Google Compute Cloud](#gcp)

## Overview <a id="overview"></a>

This project includes a number of Amalgam8 sample programs, and a preconfigured environment to allow
you to easily run, build, and experiment with the provided samples.

The following samples are available for Amalgam8:

* [Helloworld](https://github.com/amalgam8/examples/blob/master/apps/helloworld/README.md) is a single microservice app that demonstrates how to route traffic to different versions of the same microservice
* [Bookinfo](https://github.com/amalgam8/examples/blob/master/apps/bookinfo/README.md) is a multiple microservice app used to demonstrate and experiment with several Amalgam8 features

There is an end-to-end
[test & deploy demo](https://github.com/amalgam8/examples/blob/master/demo-script.md)
that you can try out on your local vagrant box.

In addition, the scripts are generic enough that you can easily deploy
them to any kubernetes based environment such as OpenShift, Google Cloud
Platform, Azure, etc. See the last section of this README for details on
how to deploy Amalgam8 on Google Cloud Platform.

## Amalgam8 with Kubernetes - local environment <a id="local-k8s"></a>

The repository's root directory includes a Vagrant file that provides an environment with everything installed and ready to build/run the samples:

* [Go](http://golang.org/)
* [Docker](http://www.docker.com/)
* [Kubernetes](http://kubernetes.io/)
* [Amalgam8 CLI](https://github.com/amalgam8/controller/tree/master/cli)
* [Gremlin SDK](https://github.com/ResilienceTesting/gremlinsdk-python)

To get started, install a recent version of [Vagrant](https://www.vagrantup.com/downloads.html) and then if you just want to run the demos, see the highlights, and kick the tires,
follow the instructions below. If you'd like to also be able to better understand the APIs, change and compile the code, or build the images,
refer the [Developer Instructions](https://github.com/amalgam8/examples/blob/master/development.md) instead.

1. Clone the Amalgam8 repos and start the vagrant environment.

    ```bash
    git clone git@github.com:amalgam8/examples.git
    
    cd examples
    vagrant up
    vagrant ssh

    cd $GOPATH/src/github.com/amalgam8
    ```
    
    *Note:* If you stopped a previous Vagrant VM and restarted it, Kubernetes might not run correctly. If you have problems, try uninstalling Kubernetes by running the following commands: 
      
    ```
    sudo examples/uninstall-kubernetes.sh
    ```
    
    Then re-install Kubernetes, by running the following command:
    
    ```
    sudo examples/install-kubernetes.sh
    ```

2. Start the local control plane services (registry and controller) by running the following commands:

    ```
    examples/controlplane/run-controlplane-local.sh start
    ```

3. Run the following command to confirm the control plane is working:

    ```bash
    a8ctl service-list
    ```

    The command shouldn't return any services, since we haven't started any yet, 
    but if it returns the follwoing empty table, the control plane servers (and CLI) are working as expected:
    
    ```
    +---------+-----------------+-------------------+
    | Service | Default Version | Version Selectors |
    +---------+-----------------+-------------------+
    +---------+-----------------+-------------------+
    ```
    
    You can also access the registry at http://192.168.33.33:5080 from the host machine
    (outside the vagrant box), and the controller at http://192.168.33.33:31200.
    To access the control plane details of tenant *local*, access
    http://192.168.33.33:31200/v1/tenants/local/ from your browser.

1. Run the [API Gateway](http://microservices.io/patterns/apigateway.html) with the following commands:

    ```bash
    kubectl create -f examples/gateway/gateway.yaml
    ```
    
    Usually, the API gateway is mapped to a DNS route. However, in our local
    standalone environment, you can access it by using the fixed IP address and
    port (http://192.168.33.33:32000), which was pre-configured for the sandbox
    environment.

1. Confirm that the API gateway is running by accessing the
    http://192.168.33.33:32000 from your browser. If all is well, you should
    see a simple **Welcome to nginx!** page in your browser.

    **Note:** You only need one gateway per tenant. A single gateway can front more
    than one application under the tenant at the same time, so long as they
    don't implement any conflicting microservices.

1. Following instructions in the README for the sample that you want to run.

  (a) *helloworld* sample

  See https://github.com/amalgam8/examples/blob/master/apps/helloworld/README.md

  (b) *bookinfo* sample

  See https://github.com/amalgam8/examples/blob/master/apps/bookinfo/README.md

1. When you are finished, shut down the gateway and control plane servers by running the following commands:

    ```
    kubectl delete -f examples/gateway/gateway.yaml
    examples/controlplane/run-controlplane-local.sh stop
    ```

## Amalgam8 on IBM Bluemix <a id="bluemix"></a>

Running Amalgam8 applications on [IBM Bluemix](http://bluemix.net/) is particulary simple because the control plane services are provided as
multi-tenanted Bluemix services, making it a matter of simply configuring and then running the tenant application itself.

To run the [Bookinfo sample app](https://github.com/amalgam8/examples/blob/master/apps/bookinfo/README.md)
on Bluemix, follow the instructions below.
If you are not a bluemix user, you can register at [bluemix.net](http://bluemix.net/).

1. Deploy the following services from the [Bluemix Service catalog](https://console.ng.bluemix.net/catalog/)
    1. [Service Discovery](https://console.ng.bluemix.net/docs/services/ServiceDiscovery/index.html) - Amalgam8 Registry Service
    2. Service Proxy version 2.0 (Coming Soon) - Amalgam8 Controller Service
    3. [Message Hub](https://console.ng.bluemix.net/docs/services/MessageHub/index.html#messagehub) - Kafka (optional)
    4. Logmet/ELK ???

    *Note:* The Amalgam8 controller service is not yet available on Bluemix, so the current demo runs a local instance of
    the A8 controller in the tenant space. This will no longer be needed in the near future.

2. Configure the [envrc file](bluemix/envrc) to your environment variable values
  1. NAMESPACE should be your Bluemix registry namespace, e.g. *cf ic namespace get*
  2. REGISTRY_SVC should be the Service Discovery service instance name
  3. ...

3. Download [Docker 1.10 or later](https://docs.docker.com/engine/installation/),
  [CF CLI 6.12.0 or later](https://github.com/cloudfoundry/cli/releases),
  [IBM Container CLI extension](https://console.ng.bluemix.net/docs/containers/container_cli_ov.html#container_cli_ov),
  and the [Amalgam8 CLI](https://pypi.python.org/pypi/a8ctl/0.1.2)
  
4. [Login to IBM Bluemix container service](https://console.ng.bluemix.net/docs/containers/container_cli_ov.html#container_cli_login)
  using *cf login* and *cf ic login*

5. (Temporary - see Note: above) Deploy the A8 controller service by running [bluemix/deploy-controller.sh](bluemix/deploy-controller.sh).
    Verify that the controller is running by ...

4. Deploy the Bookinfo app by running [bluemix/deploy-bookinfo.sh](bluemix/deploy-bookinfo.sh)

5. Configure the Amalgam8 CLI.
    export A8_CONTROLLER_URL=...

6. Confirm the microservices are running

    ```
    a8ctl service-list
    ```
    
    Should produce the following output:
    
    ```
    +-------------+---------------------+
    | Service     | Instances           |
    +-------------+---------------------+
    | productpage | v1(1)               |
    | ratings     | v1(1)               |
    | details     | v1(1)               |
    | reviews     | v1(1), v2(1), v3(1) |
    +-------------+---------------------+
     ```

6. Route all traffic to version v1 of each microservice

    ```bash
    a8ctl route-set productpage --default v1
    a8ctl route-set ratings --default v1
    a8ctl route-set details --default v1
    a8ctl route-set reviews --default v1
    ```

7. Confirm the routes are set by running the following command

    ```bash
    a8ctl route-list
    ```

    You should see the following output:

    ```
    +-------------+-----------------+-------------------+
    | Service     | Default Version | Version Selectors |
    +-------------+-----------------+-------------------+
    | ratings     | v1              |                   |
    | productpage | v1              |                   |
    | details     | v1              |                   |
    | reviews     | v1              |                   |
    +-------------+-----------------+-------------------+
    ```

    Open http://192.168.33.33:32000/productpage/productpage from your browser
    and you should see the bookinfo application displayed.
  
8. Now that the application is up and running, you can try out the other a8ctl commands as described in
  [test & deploy demo](https://github.com/amalgam8/examples/blob/master/demo-script.md)
   

## Amalgam8 with Marathon/Mesos - local environment <a id="local-marathon"></a>

This section assumes that the IP address of your mesos slave where all the
apps will be running is 192.168.33.33.

1. The `run-controlplane-mesos.sh` script in the `mesos` folder sets up a
   local marathon/mesos cluster (based on Holiday Check's
   [mesos-in-the-box](https://github.com/holidaycheck/mesos-in-the-box))  and launches the controller and the
   registry as apps in the marathon framework.
   
    ```bash
    cd mesos
    ./run-controlplane-mesos.sh start
    ```

    Make sure that the Marathon dashboard is accessible at http://192.168.33.33:8080 and the Mesos dashboard at http://192.168.33.33:5050

    Verify that the controller is up and running via the Marathon dashboard.

2. Launch the API Gateway
    
    ```bash
    cat gateway.json|curl -X POST -H "Content-Type: application/json" http://192.168.33.33:8080/v2/apps -d@-
    ```

    Verify that the gateway is reacheable by accessing http://192.168.33.33:32000

3. Launch the Bookinfo application

    ```bash
    cat bookinfo.json|curl -X POST -H "Content-Type: application/json" http://192.168.33.33:8080/v2/groups -d@-
    ```
    
    Verify that the application group has been successfully launched via the marathon dashboard.

4. You can now use the `a8ctl` command line tool to set the default
    versions for various services in the Bookinfo app, do version-based
    routing, resilience testing etc. For more details, refer to the
    [test & deploy demo](https://github.com/amalgam8/examples/blob/master/demo-script.md)

## Amalgam8 on Google Cloud Platform <a id="gcp"></a>

1. Setup [Google Cloud SDK](https://cloud.google.com/sdk/) on your machine

2. Setup a cluster of 3 nodes

3. Launch the control plane services

    ```bash
    controlplane/run-controlplane-gcp.sh start
    ```

4. Locate the node where the controller is running and assign an
   external IP to the node if needed

5. Initialize the first tenant. The `run-controlplane-gcp.sh` script stores
   the JSON payload to initialize the tenant in the `TENANT_REG` environment variable.

    ```bash
    echo $TENANT_REG|curl -H "Content-Type: application/json" -d @- http://ControllerExternalIP:31200/v1/tenants'
    ```

6. Deploy the API gateway

    ```bash
    kubectl create -f gateway/gateway.yaml
    ```

    Obtain the public IP of the node where the gateway is running. This will be
    the be IP at which the sample app will be accessible.

7. You can now deploy the sample apps as described in "Running the sample
    apps" section above. Remember to replace the IP address `192.168.33.33`
    with the public IP address of the node where the gateway service is
    running on the Google Cloud Platform.

8. Visualizing your deployment with Weave Scope

    ```bash
    kubectl create -f 'https://scope.weave.works/launch/k8s/weavescope.yaml' --validate=false
    ```

    Once weavescope is up and running, you can view the weavescope dashboard
    on your local host using the following commands
  
    ```bash
    kubectl port-forward $(kubectl get pod --selector=weavescope-component=weavescope-app -o jsonpath={.items..metadata.name}) 4040
    ```
  
    You can open http://localhost:4040 on your browser to access the Scope UI.
