# demo-qualitygates-azuredevops
This repository contains information to demo quality gates on azure devops


## Set Up Azure environment

1. Create AKS (Azure Kubernetes Service) cluster with ACR (Azure Container Registry) integration. Check out https://docs.microsoft.com/en-us/azure/aks/cluster-container-registry-integration for more details. Keep note of the registry endpoint `xxx.azurecr.io`
2. Create a new project inside Azure Devops (https://dev.azure.com/)
3. Import this repository ![Import Repo](img/ado-importrepo.png)
4. Once the import has been completed, go to `Project Settings` (bottom-left side of the screen)
5. Navigate to `Service connections` and add the following. You can select the cluster and registry from your subscription created earlier
   * Kubernetes. Name is free to chose, but using `AKS` will make it easier
   ![K8s Service Connection](img/ado-serviceconnk8s.png)
   * Docker Registry. Name is free to chose, but using `ACE Registry` will make it easier
   ![Docker Registry Connection](img/ado-serviceconnregistry.png)
6. Connect to the kubernetes cluster (can be via Cloud Shell) and install the `ingress-nginx` Ingress controller (https://kubernetes.github.io/ingress-nginx/deploy/)
7. Get the external ip of the `ingress-nginx-controller`, e.g. using the command `kubectl -n ingress-nginx get svc ingress-nginx-controller`
8. Deploy the `OneAgent Operator` on the kubernetes cluster where the application will be deployed
9.  Deploy Keptn Quality Gates on the kubernetes cluster (or elsewhere) and ensure it is exposed. Also install the [dynatrace-service](https://github.com/keptn-contrib/dynatrace-service), [dynatrace-sli-service](https://github.com/keptn-contrib/dynatrace-sli-service) and jmeter-service Note the Keptn Endpoint, Keptn API token and Keptn Bridge URL. Check out [Keptn documentation](https://keptn.sh/docs/0.7.x/operate/install/) for more info
10. Open the `azure-pipelines.yml` file (this can be done in the Azure DevOps UI) and fill in the variables:
```
variables:
- name: KEPTN_ENDPOINT
  value: 'https://xxx.nip.io/api'
- name: KEPTN_BRIDGE
  value: 'https://xxx.nip.io/bridge'
- name: KEPTN_API_TOKEN
  value: xxx
- name: KEPTN_PROJECT # can be chosen freely
  value: 'simplenode-azure'
- name: KEPTN_SERVICE # can be chosen freely
  value: 'simplenodeservice'
- name: KEPTN_STAGE # can be chosen freely
  value: 'staging'
- name: KEPTN_MONITORING
  value: 'dynatrace'
- name: SIMPLENODE_VERSION
  value: '1'
- name: APP_NAME
  value: 'simplenodeservice'
- name: INGRESS_ENDPOINT # this is the ingress for the simplenode application
  value: 'xxx.xxx.xxx.xxx.nip.io'
- name: REGISTRY_NAME
  value: 'ACE Registry' # as configured in the service connection in step 5
- name: REGISTRY # endpoint 
  value: 'aceegistry.azurecr.io'
- name: AKS_NAME # as configured in the service connection in step 5
  value: 'AKS'
```

11. Save and commit the changes
12. Go to the `Pipelines` section and click on `Create Pipeline`, Select `Azure Repos Git` and select your created project repo.
13. This will open the `azure-pipelines.yml` file with the changes made above.
14. Click on `Run` on the top-right of the screen. The Pipeline will now start.
15. Verify in Dynatrace and Keptn Bridge after the run is complete
16. To change the simplenodeservice behavior, set the `SIMPLENODE_VERSION` variable to 1(fast), 2(slower), 3(fast), 4(slow)