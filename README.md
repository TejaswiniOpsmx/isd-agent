# stormdriver-agent installation instructions

## What you need
1. A kubenetes cluster with minimum of 16GB RAM. 32GB or more may be required depending on the number of accounts configured
2. Internet accesss, specifically with access github.com and quay.io
3. A laptop with git and kubeconfig that allows connecting to the kubernetes cluster where the agent is installed

## Installation steps
### Install clouddriver and redis: 
- Clone this repo: `git clone https://github.com/OpsMx/isd-agent-v2.git`
- `cd isd-agent` 
- Edit values.yaml to update the AWS, GCP and other cloud accounts and artifacts that are not accessible from the internet
  1. Actual values for config can be found from an existing spinnaker. Contact Opsmx Support for help, if required
  2. Dynamic accounts can be enabled by uncommenting the "dynamicAccount:" tag removing the "{}" as the value and updating the repo details
- `helm install agent . -n <AGEMT-NAMAESPACE-IN_AGENT-CLUSTER> --debug`
- Create agent in ISD UI using the name "opsmx-agent" (do not use any other name)
- Download agent manifest
  1. Login the ISD UI
  2. Go to setting->agent
  3. If "opsmx-agent" is already there, click on the 3 vertical dots on the far right, edit-> Download Manifest
  4. If "opsmx-agent" is not there, click "New agent" and create one, save ->Download Manifest
- The agent is designed to run in the "default" namespace. If different, Search the downloaded manifest for "namespace:" and edit it, to update it to the namespace you are installing the agent in (e.g. opsmx-agent)
- **Create services file using opsmx-agent-services.yaml in this repo**: `kubectl apply -f opsmx-agent-services.yaml`
  1. OPTIONALLY, add kubenetes and jenkins accounts in the services.yaml and execute the "apply" command 
- Apply the agent configuration using: kubectl apply -f opsmx-agent.yaml -n <NAMAESPACE>

At this point, 3 pods should be running (opsmx-agent, spin-clouddriver and redis). Check by using; 
`kubectl get po -n <NAMAESPACE>`
  
The agent should connect to the controller. Check by logging into the ISD-UI->settings->agents. The agent should status as "Connected" (green).
  
  
### Test the Accounts
- Login to the ISD UI
- Create your application or select "sampleapp" that is already there
- Execute a pipeline that does a deployment

# Troubleshooting
## agent pod refuses to connect to 
1. Check using the logs of the agent using:
`kubectl logs -f <AGENT-POD-NAME> -n <NAMAESPACE>`

The logs should contain:  
`2022/09/02 15:09:13 version: v3.4.2, hash: 76135cc2fa2012cccb5106c5f4560d5798212821, buildType: release {"level":"info","ts":1662131353.2951057,"caller":"forwarder-agent/main.go:141","msg":"agent starting","version":"version: v3.4.2, hash: 76135cc2fa2012cccb5106c5f4560d5798212821, buildType: release","os":"linux","arch":"amd64","cores":8} {"level":"info","ts":1662131353.2960882,"caller":"forwarder-agent/main.go:177","msg":"config","controllerHostname":"controller-sdagent.opsmx.net:9001"} {"level":"info","ts":1662131353.296337,"caller":"serviceconfig/endpoints.go:99","msg":"adding endpoint","endpointType":"clouddriver","endpointName":"agent-clouddriver","endpointConfigured":true}`

If there is a message that says "context...timeouted", that means that it is unable to reach the controller. Check the internet connection

2. The agent pod goes into Error/Crashloop state. This means that the *services.yaml contains an error OR has not been applied. The agent-log should show what the issues is.

## The accounts do not show up in Spinnaker
1. Please wait a few minutes as it takes some time for the accounts/changes to be reflected in the UI
2. Check the clouddriver logs to ensure that persmissions/access is correct and that there are no errors. A stack-trace talking about "unable to fetch metadata" is normal and does not affect the functionality
3. Contact Opsmx Support  
  
# Upcoming improvements roadmap
1. ISD-UI assistence for configuring accounts and artifacts in values.yaml
2. ISD-UI to provide the following:
  - Sample config file (opsmx-agent-services.yaml)
  - agent-namespace option: generate the agent-yaml WITH the correct agent namespace
3. Agent account configuration from ISD-UI
