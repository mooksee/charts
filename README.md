# Hazelcast Helm Charts with PCF Tile

This branch contains the files and the description needed to create and use PCF Tile generated from Hazelcast Enterprise Helm Chart

## Generating PCF Tile

1. Check the version of Hazelcast Enterprise and Management Center images in `stable/hazelcast-enterprise/values.yaml` and modify if needed
2. Download Hazelcast Enterprise and Management Center Docker images to `stable/hazelcast-enterprise/images`

        $ cd stable/hazelcast-enterprise
        $ mkdir images && cd images
        $ docker pull hazelcast/hazelcast-enterprise:${HZ_VERSION}
        $ docker save hazelcast/hazelcast-enterprise:${HZ_VERSION} -o hazelcast-enterprise-${HZ_VERSION}.tgz
        $ docker pull hazelcast/management-center:${MC_VERSION}
        $ docker save hazelcast/management-center:${MC_VERSION} -o management-center-${MC_VERSION}.tgz

3. Build PCF Tile by executing (from the `tiles/hazelcast-enterprise` directory)

        $ tile build

## Installing PCF Tile

1. Open PCF Ops Manager
2. Click "Import a Product" and select the generated `*.pivotal` file
3. Add Product and click on it to complete the configuration
4. In the tab "Assign AZs and Networks", choose Network "services-1" and click "Save"
5. In the tab "Kubernetes Cluster", fill all the fields (if you have your `kubectl` context configured, you can find all of them in `~/.kube/config`)
   * Cluster CA Cert: `cluster.certificate authority data` base64-encoded (you can use [this Base64 encoder](https://www.base64decode.org/))
   * Cluster URL: `cluster.server`
   * Cluster Token: `user.token`
6. Click "Save" and go back to the "Installation Dashboard"
7. Click "Apply changes" (or "Review Pending Changes" if available for faster processing)

## Using PCF Tile

1. Login with your CF CLI tool

        $ cf login

2. Check that your PCF Tile is installed

        $ cf marketplace
		Getting services from marketplace in org system / space system as admin...
		OK

		service                           plans                     description
		hz-ent                            default                   Hazelcast IMDG Enterprise is the most widely used in-memory data grid with hundreds of thousands of installed clusters around the world. It offers caching solutions ensuring that data is in the right place when it’s needed for optimal performance.

3. Prepare the input parameters in the `values.json` file. You can override any of the Helm Chart `values.yaml` parameters. The minimal `values.json` specifies the Hazelcast License as follows.

        {
          "hazelcast": {
            "licenseKey": "<hazelcast_license_key>"
          }
        }

4. Start the Hazelcast cluster

        $ cf create-service hz-ent default test -c values.json

5. The service is started

        $ cf services
        Getting services in org system / space system as admin...

        name    service        plan      bound apps   last operation
        test    hz-ent         default                create succeeded

## Known Issues

Currently, the PCF Tile from Helm Chart is limited and the following features do not work:
 * Binding Kubernetes-based service (which makes it unusable in the PCF environment); Pivotal plans to implement this feature
 * Management Center persistence (because Helm tool from PCF does not accept `runAsUser` and persistent volume is mounted with root ownership only)

## Troubleshooting

#### Installing PCF Tile

If you have problems installing the tile, please check that the following properties are **unique** and **short** (max. 8 chars):
 * `name` in `Chart.yaml`
 * `name` in `tile.yml`

#### Creating Service

If you have problems while installing a service, you can debug it with the following steps:
1. Configure your `kubectl` tool to access the Kubernetes cluster
2. Check your namespace `$ kubectl get namespaces`
3. Check your Hazelcast PODs `$ kubectl get all -n <namespace>`
