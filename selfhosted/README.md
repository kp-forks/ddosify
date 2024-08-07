<div align="center">
    <img src="https://raw.githubusercontent.com/getanteon/anteon/master/assets/anteon-logo-db.svg#gh-dark-mode-only" alt="Anteon logo dark" width="336px" /><br />
    <img src="https://raw.githubusercontent.com/getanteon/anteon/master/assets/anteon-logo-wb.svg#gh-light-mode-only" alt="Anteon logo light" width="336px" /><br />
</div>

<h1 align="center">Anteon Self Hosted (formerly Ddosify): Effortless Kubernetes Monitoring and Performance Testing</h1>

<p align="center">
<img src="https://raw.githubusercontent.com/getanteon/anteon/master/assets/anteon_service_map_filtered.png" alt="Anteon Kubernetes Monitoring Service Map" />
<i>Anteon (formerly Ddosify) detects high latency service calls on your K8s cluster. So you can easily find the root service causing the problem.</i>
</p>

This README provides instructions for installing and an overview of the system requirements for Anteon Self-Hosted. For further information on its features, please refer to the ["What is Anteon"](https://github.com/getanteon/anteon/#what-is-anteon) section in the main README, or consult the complete [documentation](https://getanteon.com/docs/performance-testing/test-suite/).

<a href="https://aws.amazon.com/marketplace/pp/prodview-mwvnujtgjedjy" target="_blank"><img src="https://img.shields.io/badge/Available_on_aws_marketplace-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Anteon aws marketplace deployment" /></a>&nbsp;

## Effortless Installation

✅ **Arm64 and Amd64 Support**: Broad architecture compatibility ensures the tool works seamlessly across different systems on both Linux and MacOS.

✅ **Dockerized**: Containerized solution simplifies deployment and reduces dependency management overhead.

✅ **Helm Chart**: [Helm chart](https://github.com/getanteon/anteon-helm-charts) for Kubernetes deployments.

✅ **Easy to Deploy**: Automated setup processes using Docker Compose and Helm Charts.

✅ **AWS Marketplace**: [AWS Marketplace](https://aws.amazon.com/marketplace/pp/prodview-mwvnujtgjedjy) listing for easy deployment on AWS (Amazon Web Services).

✅ **Kubernetes Monitoring**: With the help of [Anteon eBPF Agent (Alaz)](https://github.com/getanteon/alaz) you can monitor your Kubernetes Cluster, create Service Map and get metrics from your Kubernetes nodes.

## 🛠 Prerequisites

- [Git](https://git-scm.com)
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/) (`docker-compose` or `docker compose`)

**Recommended System Requirements**

- **Operating System**: macOS 10.15 Catalina or later, or Linux (Ubuntu 20.04 LTS, Debian 10, CentOS 8 or later). Windows is not supported.
- **Processor**: Quad-core CPU (4 cores) at 2.5 GHz or higher, ARM-based processors are also supported (e.g., Apple M1, M2)
- **Memory**: 8 GB RAM or more
- **Storage**: At least 5 GB of available hard drive space (preferably SSD for faster load times)

> [!NOTE]
> Only Linux and MacOS are supported at the moment. Windows is not supported.

## Deploy with Docker Compose

You can quickly deploy Anteon Self Hosted by running the following command. This script clones the Anteon repository to your `$HOME/.anteon` directory, and deploys the services using Docker Compose. Please check the [install.sh](./install.sh) file to see what it does. You can also run the commands manually by following the [Manual Installation](#-manual-installation) section. 

> [!WARNING]
> Since Docker Compose deploys all the services on the same server, it is recommended to deploy Anteon Self-hosted on a Kubernetes cluster with our [Helm chart](https://github.com/getanteon/anteon-helm-charts).

Anteon Self Hosted starts in the background. You can access the dashboard at [http://localhost:8014](http://localhost:8014). The system is started always on boot if Docker is started. You can stop the system in the [Stop/Start the Services](#-stopstart-the-services) section.

```bash
curl -sSL https://raw.githubusercontent.com/getanteon/anteon/master/selfhosted/install.sh | bash
```

## Deploy on Kubernetes (Recommended)

You can deploy Anteon Self Hosted on Kubernetes using the [Helm chart](https://github.com/getanteon/anteon-helm-charts).

## 📖 Manual Installation

### 1. Clone the repository

```bash
git clone https://github.com/getanteon/anteon.git
cd anteon/selfhosted
```

### 2. Update the environment variables (optional)

The default values for the environment variables are set in the [.env](./.env) file. You can modify these values to suit your needs. The following environment variables are available:

- `DOCKER_INFLUXDB_INIT_USERNAME`: InfluxDB username. Default: `admin`
- `DOCKER_INFLUXDB_INIT_PASSWORD`: InfluxDB password. Default: `ChangeMe`
- `DOCKER_INFLUXDB_INIT_ADMIN_TOKEN`: InfluxDB admin token. Default: `5yR2qD5zCqqvjwCKKXojnPviQaB87w9JcGweVChXkhWRL`
- `POSTGRES_PASSWORD`: Postgres password. Default: `ChangeMe`

### 3. Deploy the services

```bash
docker-compose up -d
```
### 4. Access the dashboard

The dashboard is available at [http://localhost:8014](http://localhost:8014)

### 5. Show the logs

```bash
docker-compose logs
```

## 🔧 Add New Engine

The [Anteon Engine](https://github.com/getanteon/anteon) is responsible for generating load to the target URL. You can add multiple engines to scale your load testing capabilities. 

The Anteon Self Hosted includes a default engine out of the box. To integrate additional engines, simply run a Docker container for each new engine. These engine containers will automatically register with the service and become available for use. Before adding new engines, ensure that you have enabled the distributed mode by clicking the `Unlock the Distributed Mode` button in the dashboard.

In case you have modified the default values like InfluxDB password in the `.env` file, utilize the `--env` flag in the docker run command to establish the necessary environment variables.

Make sure the new engine server can access the service server. Use the `SERVICE_ADDRESS` environment variable to specify the service server address where the [install.sh](install.sh) script was executed.

The engine server must connect to the following ports on the `SERVICE_ADDRESS`:

- `9901`: Hammer Manager service. The service server utilizes this port to register the engine.
- `6672`: RabbitMQ server. The engine server connects to this port to send and receive messages to and from the service server.
- `9086`: InfluxDB server. The engine server accesses this port to transmit metrics to the backend.
- `8333`: Object storage server. The engine server uses this port to retrieve the object files like CSV and multipart files.

The `NAME` environment variable is used to specify the name of the engine container. You can change this value to whatever you want. It is also used in the [Remove New Engine](#-remove-new-engine) section for removing the engine container.

### **Example 1**: Adding the engine to the same server

```bash
NAME=anteon_hammer_1
docker run --name $NAME -dit \
    --network anteon \
    --restart always \
    ddosify/selfhosted_hammer:1.4.3
```

### **Example 2**: Adding the engine to a different server

Set `SERVICE_ADDRESS` to the IP address of the service server. Set `IP_ADDRESS` to the IP address of the engine server.

```bash
# Make sure to set the following environment variables
SERVICE_ADDRESS=SERVICE_IP
IP_ADDRESS=ENGINE_IP
NAME=anteon_hammer_1

docker run --name $NAME -dit \
    --env SERVICE_ADDRESS=$SERVICE_ADDRESS \
    --env IP_ADDRESS=$IP_ADDRESS \
    --restart always \
    ddosify/selfhosted_hammer:1.4.3
```

You should see `mq_waiting_new_job` log in the engine container logs. This means that the engine is waiting for a job from the service server. After the engine is added, you can see it in the Engines page in the dashboard.

### **Example 3**: Adding the engine to Kubernetes

You can deploy the engine on Kubernetes using the Helm chart. Please check the [Anteon Helm chart](https://github.com/getanteon/anteon-helm-charts/tree/master/charts/anteon#add-new-engine-optional) repository for more information.

## 🧹 Remove New Engine

If you added new engines, you can remove them by running the following command. Change the docker container name `anteon_hammer_1` to the name of the engine you added.

```bash
docker rm -f anteon_hammer_1
```

## 🛑 Stop/Start the Services

If you installed the project using the [install.sh](./install.sh) script, you must first change the directory to the `$HOME/.anteon` directory before running the commands below.

```bash
cd $HOME/.anteon/selfhosted
docker compose down
```

If you want to remove the complete data like databases in docker volumes, you can run the following command. ⚠️ Warning: This will remove all the data for Anteon Self Hosted.

```bash
cd $HOME/.anteon/selfhosted
docker compose down --volumes
```

You may encounter the following error when running the `docker compose down` command if you did not [remove the engine](#-remove-new-engine) containers. This is completely fine. The network `anteon` is not removed from docker. If you do not want to see this error, you can [remove the engine](#-remove-new-engine) containers first then run the `docker compose down` command again.

```text
failed to remove network anteon: Error response from...
```

If you want to start the project again, run the script in the [Quick Start](#%EF%B8%8F-quick-start-recommended) section again. 

## Enable Kubernetes Monitoring

To monitor your Kubernetes Cluster, you need to run the [Anteon eBPF Agent (Alaz)](https://github.com/getanteon/alaz) as a DaemonSet. Refer to the [Alaz](https://github.com/getanteon/alaz) repository for more information.

## 🧩 Services Overview

| Service              | Description                                                                                       |
|----------------------|---------------------------------------------------------------------------------------------------|
| `Hammer`               | The engine responsible for executing load tests. You can add multiple hammers to scale your load testing capabilities.                                                  |
| `Hammer Manager`       | Manages the engines (Hammers) involved in load testing.                                           |
| `Backend`              | Handles Kubernetes Monitoring and Performance Testing                                                   |
| `Alaz Backend`         | Handles eBPF Agent (Alaz) Metrics, Traces and Logs                                                  |
| `InfluxDB`             | Database that stores metrics collected during testing.                                            |
| `Postgres`             | Database that preserves load test results.                                                        |
| `RabbitMQ`             | Message broker enabling communication between Hammer Manager and Hammers.                         |
| `SeaweedFS Object Storage` | Object storage for multipart files and test data (CSV) used in load tests.                        |
| `Nginx`                | Reverse proxy for backend, alaz-backend and frontend services.                                                  |
| `Prometheus`          | Collects the Kubernetes Monitoring metrics from the Backend service.                                          |

## Telemetry Data

Anteon Self Hosted collects anonymous usage data to help us improve the product. You can disable this feature by setting the `ANONYMOUS_TELEMETRY_ENABLED` environment variable to `False` of the `backend` service in the [docker-compose.yml](./docker-compose.yml) file.

```yaml
backend:
    ...
    environment:
      - ANONYMOUS_TELEMETRY_ENABLED: "False"
    ...
```

### Example Data

<details>
  <summary>Here is the example of data that we collect:</summary>

```json
{
    "k8s": {
        "pod_count": 12,
        "service_count": 3,
        "daemonset_count": 1,
        "namespace_count": 4,
        "deployment_count": 2,
        "node_count_active": 1,
        "node_count_passive": 5
    },
    "metrics": {
        "cpu_count": 16,
        "total_ram_bytes": 16664440832,
        "total_root_filesystem_bytes": 730759454720
    },
    "alaz_info": {
        "ebpf": true,
        "metrics": true
    },
    "cluster_info": {
        "name": "test-cluster",
        "alaz_version": "v0.3.4",
        "instance_names": [
            "test-instance"
        ],
        "cluster_details": {
            "k8s_version": "v1.28.4+k3s2",
            "cloud_provider": "AWS",
            "kernel_versions": [
                "6.2.0-39-generic"
            ]
        }
    }
}
```
</details>



## 📝 License

Anteon Self Hosted is licensed under the AGPLv3: https://www.gnu.org/licenses/agpl-3.0.html
