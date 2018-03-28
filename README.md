# H2O + Kubeflow Integration

This is a project for the integration of H2O.ai and Kubeflow. The integration of H2O and Kubeflow is an extremely powerful opportunity, as it provides a turn-key solution for easily deployable and highly scalable machine learning applications, with minimal input required from the user.

#### Kubeflow
[Kubeflow](https://github.com/kubeflow/kubeflow) is an open source project managed by Google and built on top of their Kubernetes engine. It is designed to alleviate some of the more tedious tasks associated with machine learning. Kubeflow helps orchestrate deployment of apps through the full cycle of development, testing, and production, and allows for resource scaling as demand increases.

#### H2O 3
[H2O 3’s](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/index.html) goal is to reduce the time spent by data scientists on time-consuming tasks like designing grid search algorithms and tuning hyperparameters, while also providing an interface that allows newer practitioners an easy foothold into the machine learning space.

#### Contents
This repository contains all the necessary components for deploying H2O 3 on Kubeflow

Dockerfiles:
- Dockerfile.h2o3 -- dockerfile to build a docker image of H2O 3. This image is a standard deployment of H2O 3, and is readily available [here](https://github.com/h2oai/h2o-3/blob/master/Dockerfile) as well.
- Dockerfile.h2o3notebook -- dockerfile to build the Kubeflow Jupyterhub Notebook with H2O 3 packages. This requires the following scripts:
  - jupyter_notebook_config.py
  - start-singleuser.sh
  - start-notebook.sh
  - start.sh

kubeflow:
- directories [argo, core, driverless, h2o3, seldon, tf-job, tf-serving] contain the manifests required to deploy their corresponding applications.
  - core -- manifests required for deploying core kubeflow components
  - h2o3 -- manifests required for deploying H2O 3 components
- registry.yaml -- manifest file that declares ksonnet packages available within the directory.


#### Quick Start
Complete steps to deploy Kubeflow can be found in their [User Guide](https://github.com/kubeflow/kubeflow/blob/master/user_guide.md)

You will also need [ksonnet](https://ksonnet.io) and [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) command line tools.

- Create a Kubernetes cluster. Either on-prem or on Google Cloud
- Run the following commands to setup your ksonnet app (how you deploy Kubeflow)

```bash
# create ksonnet app
ks init <my_ksonnet_app>
cd <my_ksonnet_app>

# add ksonnet registry to app containing all the component manifests
ks registry add kubeflow <this_github_repo/kubeflow>
ks pkg install kubeflow/core
ks pkg install kubeflow/tf-serving
ks pkg install kubeflow/tf-job
ks pkg install kubeflow/h2o3

# deploy core Kubeflow componentes
kubectl create namespace kubeflow
ks generate core kubeflow-core --name=kubeflow-core --namespace=kubeflow
ks env add <my_environment_name>

# skip this line if not on cloud deployment of Kubernetes
ks param set kubeflow-core cloud gke --env=cloud

ks apply <my_environment_name> -c kubeflow-core
```

- Deploy H2O 3 by running the following commands. <location_of_docker_image> is probably a Google Container Repository with the format `gcr.io/<my_project>/<h2o3_image>:version`:

```bash
ks prototype use io.ksonnet.pkg.h2o3 h2o3 \
--name h2o3 \
--namespace kubeflow \
--model_server_image <location_of_docker_image>
```
- run `kubectl get svc -n kubeflow` to find the External IP address.
- Open a jupyter notebook on a local computer that has H2O installed locally.

```python
import h2o
h2o.init(port="<External IP address>", port=54321)
```
- You can now follow the steps for running H2O 3 AutoML that can be found [here](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/automl.html)
