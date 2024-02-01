<br />
<div align="center">
  <a href="https://digitalocean.com/">
    <img src="./assets/DO_Logo-Blue.png" alt="Logo" >
  </a>

<h3>Droplet cluster autoscaling allows a developer to create a cluster of Droplets that automatically scales based on load. To use cluster autoscaling, a developer creates a base Droplet, takes a snapshot of the Droplet and adds the Droplet to a load balancer. Clones of the base Droplet can be automatically added or removed from the cluster as load changes.</h3>
</div>

- [Architecture diagram](#architecture-diagram)
- [How to use](#how-to-use)
- [Environment variables](#environmental-variables)
- [Deploy the function](#deploy-the-function)
- [Benchmarks](#benchmarks)

## Introduction

- Droplet cluster autoscaling is one file of code called [“scale.go”](https://github.com/DO-Solutions/droplet-autoscale/blob/main/packages/autoscale/scale/scale.go) that can run as a scheduled or standalone function. It must be passed configuration parameters including the droplet tag for the cluster, load balancer name, a DigitalOcean API key, etc. and an operation to perform.

The operation to perform (op) must be one of:

* **info** return internal configuration information about the cluster
* **status** return the status of the cluster including load metrics
* **up** scale the cluster up manually by adding a droplet
* **down** scale the cluster down manually by deleting the last droplet added
* **auto** autoscale the cluster up or down based on load metrics

Example:

* <.. function url path..>/scale?op=info

### About DigitalOcean Droplets

> [DigitalOcean Droplets](https://www.digitalocean.com/products/droplets) are Linux-based virtual machines (VMs) that run on top of virtualized hardware. Each Droplet you create is a new server you can use, either standalone or as part of a larger, cloud-based infrastructure.

### About DigitalOcean Functions

> [Functions](https://www.digitalocean.com/products/functions) are blocks of code that run on demand without the need to manage any infrastructure. Develop on your local machine, test your code from the command line (using doctl), then deploy to a production namespace or App Platform — no servers required.

## Architecture diagram
<!-- image
<img src="./assets/x.png" alt="Architecture diagram" width="800">
 -->
### Prerequisites

1. A DigitalOcean account ([Log in](https://cloud.digitalocean.com/login))
2. doctl CLI - [How to install](https://docs.digitalocean.com/reference/doctl/how-to/install/)
3. To use doctl with our serverless Functions product, you must first [install a software extension](https://docs.digitalocean.com/reference/doctl/how-to/install/#step-5-install-serverless-functions-support-optional), then use it to connect to the development namespace.

### Additionally you will need:

1. A Base Droplet, this is the first Droplet that will be used for scaling
2. A Snapshot of an image you want to scale. To use droplet autoscaling, you must take a snapshot of your base droplet and that snapshot will be the system image used by all clone droplets. The snapshot should have server(s) that automatically start when the new system boots.

## How to use

### Deploy a DigitalOcean Droplet

<details>

### How to create a Droplet using the DigitalOcean CLI

To create a Droplet via the command line, follow these steps:

1. [Install `doctl`](https://docs.digitalocean.com/reference/doctl/how-to/install/), the DigitalOcean command-line tool.

2. [Create a personal access token](https://docs.digitalocean.com/reference/api/create-personal-access-token/), and save it with `doctl.`

3. Use the token to grant `doctl` access to your DigitalOcean account.

    ```bash
    doctl auth init
    ```

4. Finally, create a Droplet with `doctl compute droplet create`.

    ```bash
    doctl compute droplet create <droplet-name>... [flags]
    ```

- `base-image`` is the name of our Droplet
- `cluster_tag` is the name of the cluster tag

</details>

### Create a snapshot to be used as Droplet template image

### Tag your Droplets

### Create a Load Balancer

### Deploy the function

1. Create a Functions namespace

    ```bash
    doctl serverless namespaces create --label "scale-ns" --region "lon1"
    ```

2. Connect to your namespace

    ```bash
    doctl serverless connect
    ```

3. Clone the function from this repo `git clone https://github.com/DO-Solutions/droplet-autoscale`

4. Deploy the function

    ```bash
    doctl serverless deploy droplet-autoscale
    ```

### Environmental Variables

### The following environment variables are required:

* **API_TOKEN** a DigitalOcean API token
* **BASE_DROPLET** the name of the base droplet to scale
* **SNAPSHOT** To use droplet autoscaling, you must take a snapshot of your base droplet and that snapshot will be the system image used by all clone droplets. The snapshot should have server(s) that automatically start when the new system boots. The snapshotName is the name of this base droplet snapshot.
* **CLUSTER_TAG** the droplet tag for the cluster. The base droplet must have this tag. All clones that are created will have this tag on them.
* **CLONE_PREFIX** the name of clone droplets is the dropletNamePrefix and the number of the clone. The first clone will be <prefix>01, the second <prefix>02, etc. An example of a droplet name prefix is “auto”
* **LOAD_BALANCER** Droplet autoscaling requires a load balancer. You can find the DigitalOcean load balancer under the Network section of the DigitalOcean user-interface. The base droplet must be added to the load balancer. Clone droplets will automatically be added and removed from the load balancer cluster as the system scales up and down. The loadBalancerName is the name of this load balancer.

### The following environment variables are optional:
    
* **MAX_DROPLETS_IN_CLUSTER** Defaults to 5. The maximum number of droplets in the cluster inclusive of the base droplet. The system will not scale up beyond this number of droplets.
* **SCALING_INTERVAL** Defaults to 120 seconds. After a clone droplet is added or removed, the system will stop autoscaling up or down until this interval has passed. The system adds or removes clones automatically based on load metrics. These metrics take some time to stabilize after a clone is added or removed. The scalingInterval is the amount of time it takes the system to stabilize, in seconds, after a droplet is added or removed and should not only allow the system to stabilize but also account for the time it takes the new load metrics from the droplets to be available.
* **SCALE_UP_WHEN_LOAD** When the average CPU load metric is greater than this value, the system will recommend scaling up.
* **SCALE_UP_WHEN_FREE_MEMORY** When the average droplet free memory is below this value, in MB, the system will recommend scaling up.
* **SCALE_UP_WHEN_OUTBOUND_BANDWIDTH** When the average droplet bandwidth, in MBps, is greater than this value, the system will recommend scaling up.
* **SCALE_DOWN_WHEN_LOAD** When the average CPU load metric is less than this value and all other SCALE_DOWN metric values are within range, the system will recommend scaling down.
* **SCALE_DOWN_WHEN_FREE_MEMORY** When the average droplet free memory is above this value and all other SCALE_DOWN metric values are within range, in MB, the system will recommend scaling down.
* **SCALE_DOWN_WHEN_OUTBOUND_BANDWIDTH** When the average droplet bandwidth, in MBps, is less than this value and all other SCALE_DOWN metric values are within range, the system will recommend scaling down.

## Examples

## Conclusion

In this guide, we deployed a DigitalOcean Managed Kubernetes cluster. We setup [k8s-csi-s3](https://github.com/yandex-cloud/k8s-csi-s3), which is a [GeeseFS-based](https://github.com/yandex-cloud/geesefs) CSI for mounting S3 buckets as PersistentVolumes by deploy the driver, creating a new storage class and deploying a test pod with RWX storage.

## References

Make some reference to internal DO engineering originally creating this.

<!-- CONTACT
# Contact

Jack Pearce, Solutions Engineer - jpearce@digitalocean.com

<p align="right">(<a href="#top">back to top</a>)</p>

 -->