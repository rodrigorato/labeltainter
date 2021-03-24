# labeltainter
Set Kubernetes node taints based on node or custom labels without having to configure that while bootstrapping the nodes.

Useful for managed node group setups where you don't control the nodes directly, so not able to set taints on the nodes easily.

This is what I believe to be an elegant solution given the constraints for the problem.
However, do keep in mind that this is far from ideal and should only be used when no other option is available.

## Goals
This project has the following objectives:
* Allow adding taints on Kubernetes nodes based:
    * Label key/values if key is present on nodes;
    * Label key/values if pair is present on nodes;
    * Custom label key/value combinations;
* Keep the lowest possible runtime overhead;
* Keep the solution as simple and as clean as possible;
* Log all changes made to the cluster for auditing purposes;

## Non-Goals
Taking into account the goals above, it is out of the scope of this project to:
* Implement a custom Kubernetes controller/operator to solve this problem (as it would make the solution unnecessarily complex);
* Allow for taints to be added using a _"mutation"_ on a label that is present on a node (e.g. changing label values or names on the nodes and setting that as taints);

## Setup
### Installing labeltainter on your cluster
To install it directly on your cluster you can:
```sh
$ kubectl apply -f https://raw.githubusercontent.com/rodrigorato/labeltainter/main/k8s/labeltainter.yaml
```

**Although...** You'd probably like to touch up the configuration first before installing it, so that it features the label set you want to use as taints, so go ahead and download `k8s/labeltainter.yaml` and change the configuration before applying it (or build a kustomize layer based on it if thats your thing).

### Uninstalling labeltainter from your cluster
A simple uninstallation would be:
```sh
$ kubectl delete -f https://raw.githubusercontent.com/rodrigorato/labeltainter/main/k8s/labeltainter.yaml
```

Hoewever, depending on the changes you made while installing, there might be extra steps to clean up what you've done. :)

## Architecture
We make use of a Kubernetes `DaemonSet` that tolerates all taints to make sure that our code runs on every node, and it runs with a service account (and token) that is allowed to read and patch `Node` objects on the cluster.

An init container is used to make sure this runs exactly once, when the node starts, and after the init container is done setting taints we'll keep a pause container going for minimal resource consumption.

The init container logs show exactly what changes were made, based on your configuration for `labeltainter`, as well as the cluster nodes' existing labels.

## Acknowledgements
* [Raul Gonzales](https://github.com/gonzalesraul) for the brainstorm;
* [Nuno Silva](https://github.com/nuno-silva) for simplifying the script;
* **You**, maybe? If you got something to contribute I'll definitely add you here! :)
