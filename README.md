# Skuber

Skuber is a Scala client library for [Kubernetes](http://kubernetes.io). It provides a high-level and strongly typed Scala API for remotely managing and reactively monitoring Kubernetes resources (such as Pods, Containers, Services, Replication Controllers etc.) via the Kubernetes API server.

The client supports v1.0 and v1.1 of the Kubernetes REST API.

## Sample Usage

The code block below illustrates the simple steps required to create a replicated nginx service on a Kubernetes cluster.
 
The service uses five replicated pods, each running a single Docker container of an nginx image. Each pod publishes the exposed port 80 of its container enabling access to the nginx service within the cluster.

The service can be accessed from outside the cluster at port 30001 on each cluster node, which Kubernetes proxies to port 80 on the nginx pods. 

    import skuber._
    import skuber.json.format._

    val nginxSelector  = Map("app" -> "nginx")
    val nginxContainer = Container("nginx",image="nginx").port(80)
    val nginxController= ReplicationController("nginx",nginxContainer,nginxSelector).withReplicas(5)
    val nginxService   = Service("nginx", nginxSelector, Service.Port(port=80, nodePort=30001)) 

    import scala.concurrent.ExecutionContext.Implicits.global

    val k8s = k8sInit

    val createOnK8s = for {
      svc <- k8s create nginxService
      rc  <- k8s create nginxController
    } yield (rc,svc)

    createOnK8s onComplete {
      case Success(_) => System.out.println("Successfully created nginx replication controller & service on Kubernetes cluster")
      case Failure(ex) => System.err.println("Encountered exception trying to create resources on Kubernetes cluster: " + ex)
    }

    k8s.close

See the [programming guide](docs/GUIDE.md) for more details.

The `examples` sub-project also illustrates several features. See for example the [reactive guestbook](examples/src/main/scala/skuber/examples/guestbook) example.

## Features

- Comprehensive Scala case class representations of the Kubernetes types supported by the API server; including Pod, Service, ReplicationController, Node, Container, Endpoint, Namespace, Volume, PersistentVolume, Resource, Security, EnvVar, ServiceAccount, LimitRange, Secret, Event and others
- Support for Kubernetes object, list and simple kinds
- Fluent API for building the desired specification ("spec") of a Kubernetes object to be created or updated on the server 
- Implicit json formatters for reading and writing the Kubernetes types
- Support for create, get, delete, list, update, and watch operations on Kubernetes types using an asynchronous and type-safe interface that maps each operation to the appropriate Kubernetes REST API requests. 
- Watching Kubernetes objects and kinds returns Iteratees for reactive processing of events from the cluster
- Client contexts (including connection details and namespace) can be configured by a combination of system properties and a config file format based on the Kubernetes kubeconfig file YAML format
- Support for horizontal pod auto scaling (Kubernetes V1.1 beta feature)

## Build Instructions

The project consists of two sub-projects - the main Skuber client library (under the `client` directory) and an `examples` project.

A sbt build file is provided at the top-level directory, so you can use standard sbt commands to build jars for both projects or select one project to build.

## Requirements

- Java 8 (build and run time)
- sbt 0.13 (build time)
- Kubernetes v1.0 or later (run time)

Use of the newer extensions API group features ( currently supported: Scale, HorizontalPodAutoScaler) requires a v1.1 Kubernetes cluster at run time. 

## Status

The coverage of the Kubernetes API functionality by Skuber is extensive, however this is an alpha release with all the caveats that implies. 

- Testing has largely used the default configuration, which connects to a Kubernetes cluster via a kubectl proxy running on localhost:8001 and uses the default namespace. Your mileage may vary with other client configurations.
- If some functionality isn't covered by the tests and examples included in this release you should assume it hasn't been tested.
- Documentation is currently sparse - in practice a basic knowledge of Kubernetes as well as Scala experience will be required, from there the Skuber [programming guide](docs/GUIDE.md) and [examples](examples/src/main/scala/skuber/examples) should help get you up and running.
- Support of the [beta features in Kubernetes v1.1](http://blog.kubernetes.io/2015/11/Kubernetes-1-1-Performance-upgrades-improved-tooling-and-a-growing-community.html) currently includes [horizontal pod autoscaling](http://kubernetes.io/v1.1/docs/user-guide/horizontal-pod-autoscaler.html); support for other Kubernetes v1.1 [Extensions API group](http://kubernetes.io/v1.1/docs/api.html#api-groups) features such as [Daemon Sets](http://kubernetes.io/v1.1/docs/admin/daemons.html), [Deployments](http://kubernetes.io/v1.1/docs/user-guide/deployments.html), [Jobs](http://kubernetes.io/v1.1/docs/user-guide/jobs.html) and [Ingress / HTTP load balancing](http://kubernetes.io/v1.1/docs/user-guide/ingress.html) is due shortly.

## License

This code is licensed under the Apache V2.0 license, a copy of which is included [here](LICENSE.txt).