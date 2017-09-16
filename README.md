# Docker Registry BOSH Release

This is a [BOSH](http://bosh.io/) release for [Docker Registry](https://docs.docker.com/registry/), a stateless, highly scalable server side application that stores and lets you distribute Docker images.

## Disclaimer

This is **NOT** a production ready [BOSH](http://bosh.io/) release. I created it for my own experimentation and it may not be supported in the future.

If you are looking for a supported [Docker Registry](https://docs.docker.com/registry/) [BOSH](http://bosh.io/) release, please check the [bosh.io](http://bosh.io/releases) releases page or the [cloudfoundry-community](https://github.com/cloudfoundry-community) github organization.

## Table of Contents

* [Usage](https://github.com/frodenas/docker-registry-boshrelease#usage)
  * [Requirements](https://github.com/frodenas/docker-registry-boshrelease#requirements)
  * [Clone the repository](https://github.com/frodenas/docker-registry-boshrelease#clone-the-repository)
  * [Basic deployment](https://github.com/frodenas/docker-registry-boshrelease#basic-deployment)
  * [Operations files](https://github.com/frodenas/docker-registry-boshrelease#operations-files)
  * [Deployment variables and the var-store](https://github.com/frodenas/docker-registry-boshrelease#deployment-variables-and-the-var-store)
* [Contributing](https://github.com/frodenas/docker-registry-boshrelease#contributing)
* [License](https://github.com/frodenas/docker-registry-boshrelease#license)

## Usage

### Requirements

In order to use this BOSH release you will need:

* [BOSH CLI v2](https://bosh.io/docs/cli-v2.html)
* An already deployed [BOSH environment](http://bosh.io/docs/init.html)
* A compatible [cloud-config](http://bosh.io/docs/terminology.html#cloud-config) with a `default` option for `network` and `vm_types` (you can use the example that comes from [cf-deployment](https://github.com/cloudfoundry/cf-deployment/blob/master/bosh-lite/cloud-config.yml))

###  Clone the repository

First, clone this repository into your workspace:

```
git clone https://github.com/frodenas/docker-registry-boshrelease
cd docker-registry-boshrelease
export BOSH_ENVIRONMENT=<name>
```

### Basic deployment

To deploy a basic [Docker Registry](https://docs.docker.com/registry/) use the following command:

```
bosh -d docker-registry deploy manifests/docker-registry.yml \
  --vars-store tmp/deployment-vars.yml
```

Once deployed, look for the `registry` instance IP address:

```
bosh -d docker-registry instances
```

And the registry password (located at the `tmp/deployment-vars.yml` file or [CredHub](https://docs.cloudfoundry.org/credhub/) if it is enabled at your BOSH Director). Then login into your [Docker Registry](https://docs.docker.com/registry/):

```
docker login -u admin -p <password> <registry instance IP address>:5000
```

*NOTE: if you don't provide TLS certificates issued by a known CA (for example by letting [BOSH](http://bosh.io/) generate self-signed certificates), then you will need to add the IP address of your [Docker Registry](https://docs.docker.com/registry/) to the `insecure-registries` setting of your [Docker Engine](https://docs.docker.com/engine/) (see [Test an insecure registry](https://docs.docker.com/registry/insecure/) documentation).*

### Operations files

Additional [operations files](http://bosh.io/docs/cli-ops-files.html) are located at the [manifests/operators](https://github.com/frodenas/docker-registry-boshrelease/tree/master/manifests/operators) directory. Those files includes a basic configuration, so extra ops files might be needed for additional configuration.

Please review the op files before deploying them to check the requeriments, dependencies and necessary variables.

| File | Description |
| ---- | ----------- |
| [enable-cf-route-registrar.yml](https://github.com/frodenas/docker-registry-boshrelease/blob/master/manifests/operators/enable-cf-route-registrar.yml) | Registers `registry` as a [Cloud Foundry route](https://docs.cloudfoundry.org/devguide/deploy-apps/routes-domains.html) (under your `system domain`) |
| [enable-nginx.yml](https://github.com/frodenas/docker-registry-boshrelease/blob/master/manifests/operators/enable-nginx.yml) | Enables [nginx](https://nginx.org/) as a proxy in front of your [Docker Registry](https://docs.docker.com/registry/) instances |
| [enable-redis-cache.yml](https://github.com/frodenas/docker-registry-boshrelease/blob/master/manifests/operators/enable-redis-cache.yml) | Enables [Redis](https://redis.io/) as a cache for your [Docker Registry](https://docs.docker.com/registry/) instances |
| [mirror-docker-hub.yml](https://github.com/frodenas/docker-registry-boshrelease/blob/master/manifests/operators/mirror-docker-hub.yml) | Configures your [Docker Registry](https://docs.docker.com/registry/) as a [Docker Hub](https://hub.docker.com/) pull through cache |
| [use-azure-storage.yml](https://github.com/frodenas/docker-registry-boshrelease/blob/master/manifests/operators/use-azure-storage.yml) | Uses [Microsoft Azure Storage](https://azure.microsoft.com/en-us/services/storage/) as the [Docker Registry storage backend](https://docs.docker.com/registry/configuration/#storage) |
| [use-gcs-storage.yml](https://github.com/frodenas/docker-registry-boshrelease/blob/master/manifests/operators/use-gcs-storage.yml) | Uses [Google Cloud Storage](https://cloud.google.com/storage/) as the [Docker Registry storage backend](https://docs.docker.com/registry/configuration/#storage) |
| [use-oss-storage.yml](https://github.com/frodenas/docker-registry-boshrelease/blob/master/manifests/operators/use-oss-storage.yml) | Uses [Aliyun Object Storage Service](https://www.alibabacloud.com/product/oss) as the [Docker Registry storage backend](https://docs.docker.com/registry/configuration/#storage) |
| [use-s3-storage.yml](https://github.com/frodenas/docker-registry-boshrelease/blob/master/manifests/operators/use-s3-storage.yml) | Uses [Amazon S3](https://cloud.google.com/storage/) as the [Docker Registry storage backend](https://docs.docker.com/registry/configuration/#storage) |
| [use-swift-storage.yml](https://github.com/frodenas/docker-registry-boshrelease/blob/master/manifests/operators/use-swift-storage.yml) | Uses [OpenStack Swift](https://docs.openstack.org/swift/latest/) as the [Docker Registry storage backend](https://docs.docker.com/registry/configuration/#storage) |

### Deployment variables and the var-store

Some operators files requires additional information to provide environment-specific or sensitive configuration such as various credentials. To do this in the default configuration, we use the `--vars-store`. This flag takes the name of a `yml` file that it will read and write to. Where necessary credential values are not present, it will generate new values based on the type information stored at the different deployment files. Necessary variables that BOSH can't generate need to be supplied as well.
See each particular op files you're using for any additional necessary variables.

See also the [BOSH CLI documentation](http://bosh.io/docs/cli-int.html#value-sources) for more information about ways to supply such additional variables.

## Contributing

Refer to [CONTRIBUTING.md](https://github.com/frodenas/docker-registry-boshrelease/blob/master/CONTRIBUTING.md).

## License

Apache License 2.0, see [LICENSE](https://github.com/frodenas/docker-registry-boshrelease/blob/master/LICENSE).
