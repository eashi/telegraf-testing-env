# Setting up AKS Testing Environment

- Create an AKS cluster
- Create an Azure Container Registry (ACR)
- Make sure that you have the right number of nodes
- Mark one of the nodes for Telegraf-only deployment, for example if you want to makr the node `aks-agentpool-65819783-1` then you do that by two stesps:
    - invoke the command `kubectl taint nodes aks-agentpool-65819783-1 workload=telegraf:NoSchedule`. This will make sure that no workload is scheduled on this node unless the worload has the same `teleration` of `workload=telegraf:NoSchedule`.
    - inovoke the command `kubectl label node aks-agentpool-65819783-1 workload=telegraf`, this will add the label `workload=telegraf` to the node. Then once we want to create the Telegraf deployment (coming below) we use the `nodeSelector` configuration with the same label to make sure that the deployment is scheduled on that node sepcifically.
- The folder `sample-helm-chart` is a folder borrowed from the repository https://github.com/microsoft/mssql-docker, but we modified it so that:
    - The secret in `templates/secret.yaml` is unique to the release by adding `{{ .Release.Name }}` to the end of the name of the secret.
    - Do the same as above to the configMap in `templates/mssqlconfig.yaml`.
    - Make sure to replace the references to the names of the secret and the config above in the `templates/deployment.yaml` file.
    - Remove all the `volumes` and the `volumeMounts` in `templates/deployment.yaml` except for the `mssql-config-volume` volume. We do this because we are just measuring the metrics part of the server, we don't need to persist any data.
    - In the `values.yaml` change the service type to be “ClusterIP” as we don’t want to expose the SQL servers to the internet.
- From any Telegraf repo (where you want to create the image from):
    - checkout the branch you want create an image from.
    - From the terminal in the root path Telegraf, invoke the command `make docker-image`. This will build a docker image tagged with the current commit SHA (make sure you have Docker installed).
    

