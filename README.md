# ISD-agent installation instructions

## What you need
1. ISD Already installed and configured. This is typically the Saas Instance provided by OpsMx. You will need login credentials to the instance.
2. A kubenetes cluster with minimum of 4CPU/16GB RAM. Recommend 8CPU/32GB or more depending on the number of accounts and cluster sizes
3. Direct Internet accesss, specifically with access to github.com and quay.io and private access to accounts and artifacts with in your corporate network
4. A laptop with git and kubectl (along with kubeconfig) commands that allows connecting to the kubernetes cluster where the agent is expected to be installed

## Installation steps
### Install clouddriver and redis: 
- Clone this repo: `git clone https://github.com/OpsMx/isd-agent-v2.git`
- Change to repository folder: `cd isd-agent` 
- Edit `values.yaml` to update the AWS, GCP and other cloud accounts and artifacts that are not accessible from the internet
  - The README in samples folder provides additional instructions, samples and documentation. Contact Opsmx Support for help, if required
- Using the target namespace for the agent, install via Helm: `helm install agent . -n <AGENT-NAMESPACE> --debug`
- Move to the root of the cloned repo: `cd ..` 
## Create agent in ISD UI using the name `opsmx-agent`
- Download agent manifest from ISD
  1. Login the ISD UI
  2. Go to setting->agent
  3. If your agent already exists:
      - click on the 3 vertical dots on the far right
      - edit
      - Download Manifest
  4. If your agent is not there:
      - click "New agent" and create one, give a name that is short-form of your company name, please add description as appropriate e.g contact information
      - save
      - Download Manifest
- The agent is designed to run in the namespace named `default`. If using a different namespace, search the downloaded manifest for `namespace:` and edit it to update it to the namespace you are installing the agent in (e.g. opsmx-agent).
  - Alternatively, change the "ClusterRole"/"ClusterRoleBinding" to "Role"/"RoleBinding" and delete the "namespace:" line, if this what you prefer
- **Create services file using opsmx-agent-services.yaml in this repo**: `kubectl apply -f opsmx-agent-services.yaml -n AGENT-NAMESPACE`
   - Please follow the instructions provided in the sample file in the root folder of this repo
- Apply the agent configuration using: `kubectl apply -f <downloaded-agent-manifest>.yaml -n AGENT-NAMESPACE`

At this point, 3 pods should be running (opsmx-agent-AGENT_NAME, spin-clouddriver and redis). Check by using 
`kubectl get po -n AGENT-NAMESPACE`
  
The agent should connect to the controller. Check by logging into the ISD-UI->settings->agents. The agent should status as "Connected" (green).
  
### Test the Accounts
- Login to the ISD UI
- Create your application or select "sampleapp" that is already there
- Execute a pipeline that does a deployment

# Troubleshooting
## Agent pod refuses to connect 
1. Check using the logs of the agent using:
`kubectl logs -f <AGENT-POD-NAME> -n <NAMAESPACE>`

The logs should contain:
```
2022/09/02 15:09:13 version: v3.4.2, hash: 76135cc2fa2012cccb5106c5f4560d5798212821, buildType: release {"level":"info","ts":1662131353.2951057,"caller":"forwarder-agent/main.go:141","msg":"agent starting","version":"version: v3.4.2, hash: 76135cc2fa2012cccb5106c5f4560d5798212821, buildType: release","os":"linux","arch":"amd64","cores":8} {"level":"info","ts":1662131353.2960882,"caller":"forwarder-agent/main.go:177","msg":"config","controllerHostname":"controller-sdagent.opsmx.net:9001"} {"level":"info","ts":1662131353.296337,"caller":"serviceconfig/endpoints.go:99","msg":"adding endpoint","endpointType":"clouddriver","endpointName":"agent-clouddriver","endpointConfigured":true}
```

If there is a message that says "context...timeouted", that means that it is unable to reach the controller. Check the internet connection between the agent and the host cluster.

# Troubleshooting

## The agent pod goes into Error/Crashloop state
1. This means that the *services.yaml contains an error OR has not been applied. The agent-log should show what the issues is.

## The accounts do not show up in Spinnaker
1. Please wait a few minutes as it takes some time for the accounts/changes to be reflected in the UI
2. Check the clouddriver logs to ensure that persmissions/access is correct and that there are no errors. A stack-trace talking about "unable to fetch metadata" is normal and does not affect the functionality
  - It should show the account names (e.g my-aws-acc) in the logs as "Caching....ACCOUNT NAME" e.g: `my-k8s-acc/KubernetesCoreCachingAgent[1/1]: grouping statefulSet has 1 entries and 5 relationships` 
  - If there are any errors "unable to access" or "unauthorized", please check your authentication settings
3. Contact Opsmx Support  
  
## Upcoming improvements roadmap
1. ISD-UI assistence for configuring accounts and artifacts in values.yaml
2. ISD-UI to provide the following:
    - Sample config file (opsmx-agent-services.yaml)
    - agent-namespace option: generate the agent-yaml WITH the correct agent namespace
3. Agent account configuration from ISD-UI
