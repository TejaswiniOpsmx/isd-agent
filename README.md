# stormdriver-agent installation instructions

## What you need
1. A kubenetes cluster with minimum of 16GB RAM. 32GB or more may be required depending on the number of accounts configured
2. Internet accesss, specifically to access github.com and quay.io
3. A laptop with git and kubeconfig that allows connecting to the kubernetes cluster where the agent is installed

## Installation steps
### Install clouddriver and redis: 
- Clone this repo: `git clone https://github.com/OpsMx/isd-agent-v2.git`
- `cd isd-agent` 
- Edit values.yaml to update the AWS, GCP and other cloud accounts and artifacts that are not accessible from the internet
  1. Actual values for config can be found from existing spinnaker. Contact Opsmx Support for help, if required
  2. Dynamic accounts can be enabled by uncommenting the "dynamicAccount:" comments removing the "{}" as the value
- `helm install agent . -n <AGEMT-NAMAESPACE-IN_AGENT-CLUSTER> --debug`
- Create agent in ISD UI using the name "opsmx-agent" (do not use any other name)
- Download agent manifest
  1. Login the ISD UI
  2. Go to setting->agent
  3. If "opsmx-agent" is already there, click on the 3 vertical dots on the far right, edit-> Download Manifest
  4. If "opsmx-agent" is not there, click "New agent" and create one, save ->Download Manifest
- The agent is designed to run in the "default" namespace. If different, Search the downloaded manifest for "namespace:" and edit it, to update it to the namespace you are installing the agent in (e.g. opsmx-agent)
- **Create services file using opsmx-agent-services.yaml in this repo**: `kubectl apply -f opsmx-agent-services.yaml`
- Apply the agent configuration using: kubectl apply -f opsmx-agent.yaml -n <NAMAESPACE>

  
At this point, 3 pods should be running (opsmx-agent, spin-clouddriver and redis). Check by using; 
`kubectl get po -n <NAMAESPACE>`
  
The agent should connect to the controller. Check using the logs of the agent using:
`kubectl logs -f <AGENT-POD-NAME> -n <NAMAESPACE>`

The logs should contain:  
`2022/09/02 15:09:13 version: v3.4.2, hash: 76135cc2fa2012cccb5106c5f4560d5798212821, buildType: release {"level":"info","ts":1662131353.2951057,"caller":"forwarder-agent/main.go:141","msg":"agent starting","version":"version: v3.4.2, hash: 76135cc2fa2012cccb5106c5f4560d5798212821, buildType: release","os":"linux","arch":"amd64","cores":8} {"level":"info","ts":1662131353.2960882,"caller":"forwarder-agent/main.go:177","msg":"config","controllerHostname":"controller-sdagent.opsmx.net:9001"} {"level":"info","ts":1662131353.296337,"caller":"serviceconfig/endpoints.go:99","msg":"adding endpoint","endpointType":"clouddriver","endpointName":"agent-clouddriver","endpointConfigured":true}`

If there is a message that says "context...timeouted" and.or the agent pod goes into Error/Crashloop state, pLease see the troubleshooting tips at the end of this document
  
3. Configure AWS or other accounts by editing the values.yaml in agent-repo (isd-agent folder)


## Updating Stormdriver configmap
We need to set-up the route to opmsx-controller-controller1:7002 using the JWT key that is generated using an API call. The proceedure is as follows;
- Exec into oes-sapor and execute this command
curl -d '{"agentName": "opsmx-agent", "name": "agent-clouddriver", "type": "clouddriver"}' https://opsmx-controller-controller1:9003/api/v1/generateServiceCredentials --cacert /opt/opsmx/controller/ca.crt --key /opt/opsmx/controller/cert/tls.key --cert /opt/opsmx/controller/cert/tls.crt
- locate the user-name (agent-clouddriver.opsmx-agent) and password in the response (this can change but is a LONG string of random chars). Copy paste and keep it ready
- Create the URL:https://agent-clouddriver.opsmx-agent:PASSWORD-FROM-JSON-RESPONSE@opsmx-controller-controller1:9002 and update it in stormdriver-config map 

# Recommendations
1. Stormdriver configuration update (and opsmx-controller config update, that is not mentioned in this document) should not be required: **Michael has taken up the action item**
  - This avoids the complicated step of creating the URL for connecting stormdriver to the controller
2. ISD-UI should provide the following:
  - Sample config file (opsmx-agent-services.yaml, with the agent-name substituted correctly) [This avoids editing the file if no other account is to be added]
  - agent-namespace: generate the agent-yaml WITH the correct agent namespace [ This avoids editing the yaml if default namespaces is not available/used for the agent]
3. agent-service (isd-agent/templates/agent/opsmx-agent-svc.yaml) should be included in the downloaded manifest. It does not hurt even if no one is using it. For now, it is included as part of the helm-chart. That is a bad idea as we COULD include it as the agent name in values.yaml but it was already given in the ISD-UI!! So we would be asking the user for the agent-name 2 times and that is cause of frustration and errors.
