# Fleet Security Demo

This is a small demo on utilizing Fleet to automate the deployment of Neuvector and Kubewarden, a few of the security tools from Rancher.

## Prereqs

1. You need a Kubernetes cluster running Rancher, ideally 2.7.0+ for the best experience with Kubewarden.
2. While you can apply these to the `local` cluster that is running Rancher, the best experience to apply this to downstream clusters. With that said, Rancher should be managing at least one downstream cluster you want to run the security tools on.
3. (Optional) You should enable Rancher UI Extensions and install the [Kubewarden UI Extension](https://github.com/kubewarden/ui#install).

## Getting Started

1. Update the `global.cattle.url` value in [values.yaml](neuvector/core/values.yaml) to point to your Rancher URL.
2. Set your Kube context to the local cluster that is running Rancher, and from the root of this repository, run the following:
   ```bash
   kubectl apply -f kubewarden/gitrepo.yaml
   kubectl apply -f neuvector/gitrepo.yaml
   ```
3. Navigate to Rancher, and in the menu select `Continuous Delivery`. You should now have 2 repositories, both showing `0/0` clusters being ready.

## Creating a Cluster Groups

Fleet gives you the ability, through labels on clusters, to create and manage groups of clusters. This example is using Cluster Groups, so we'll need to make a Cluster Group for our clusters.

1. Navigate back to `Continuous Delivery` and select `Cluster Groups`. Click `Create` in the upper-right.
2. Type `secured` as the name, and click `Add Rule`.
3. In the boxes, enter `security` -> `in list` -> `enabled`.
4. Click `Create`.

## Labeling Clusters

Now that we have our cluster group, we need to label our cluster(s) to be part of that group.

1. Navigate back to `Continuous Delivery` and select `Clusters`.
2. Select the respective cluster you want to use, click the 3-dot button, and select `Edit Config`.
3. Click `Add Label`, and set a value of `security` -> `enabled`.
4. Click `Save`.

These labels can also be created at creation time of the cluster, so as soon as a cluster comes online, it will be applicable. You can also have cluster groups that default to all found clusters.

## Checking for Updates

Now that your cluster is part of the Cluster Group, Fleet should be begin installing the security tools.

1. Navigate back to `Continuous Delivery`.
2. Select `Git Repos`.
3. Monitor progress. Both repos should turn to `Active` after a period of time.

## Validating It Worked

Now that the security tools are installed, let's validate.

1. Under the Rancher menu's `Explore Cluster`, select your cluster. There should now be a `Neuvector` and (if you installed UI Extensions) a `Kubewarden` menu item.
2. Click `Neuvector`, then click the dialog to open Neuvector. After you accept the terms, you should now have access to Neuvector!
3. Click `Kubewarden`, and you should see that you have a Policy Server running and 2 active policies.
4. Let's validate that those policies are enforcing. Running the following, which should yield failures:
   ```bash
   # Failure due to annotations policy
   kubectl apply -f example-resources/pod.yaml

   # Failure due to istio-injection policy
   kubectl apply -f example-resources/namespace.yaml
   ```

## Clean Up

To clean up your cluster, run the following on the `local` cluster:
   ```bash
   kubectl delete -f kubewarden/gitrepo.yaml
   kubectl delete -f neuvector/gitrepo.yaml
   ```

## More Information

* [Rancher Fleet](https://fleet.rancher.io/)
* [Neuvector](https://open-docs.neuvector.com/)
* [Kubewarden](https://docs.kubewarden.io/)
* [Kubewarden Istio Policy](https://github.com/atoy3731/kubewarden-istio-policy-rust)