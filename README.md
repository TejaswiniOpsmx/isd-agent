# stormdriver-agent
Agent for stormdriver that includes clouddriver and redis

Instructions for installing the stormdriver:
1. Install ISD
- Set global.stormdriver: true in values.yaml
- ensure that forwarder.externalName is set and is reachable from the agent on port 9001
- In the Spinnaker-git-repo, ensure that we copy .hal/config and .hal/default/profiles/spinnaker.yaml from this repo/SPINNAKER-FILES
[Explanation: clouddriver is set to non-HA and routed to stormdriver:7002]
- Install spinnaker from enterprise-spinnaker, v4.0 branch [Explanation: Multiple changes:oes-gate routes clouddriver calls to stormdriver, controller is configured to route calls to front50 and fiat from remote-clouddriver]
- **Create Agent-clouddriver URL using the instructions below and update stormdriver-config CM using**: `k edit cm stormdriver-config`
- restart stormdriver

2. Install agent in a different cluster/namespace
- Install clouddriver component: a) Clone this repo, b) cd to isd-agent c) `helm upgrade --install agent . -n <AGEMT-NAMAESPACE-IN_AGENT-CLUSTER> --debug`
[ Explanation: Clouddriver is "manually configured", agent-service needs to be added (See recommendations), redis is installed]
- Create agent in ISD UI using the name "opsmx-agent" (do not use any other name)
- Download manifest
- Search the manifest for "namespace:" and edit it ("default" is the namespace already there) to update it to the namespace you are installing the agent in
- **Create services file using opsmx-agent-services.yaml in this repo**: `k apply -f opsmx-agent-services.yaml`
- Create the agent using: k apply -f opsmx-agent.yaml -n <NAMAESPACE>

At this point, agent should connect to the controller. Check using the logs of the agent.

3. Configure AWS or other accounts by editing the values.yaml in agent-repo (isd-agent folder)
- Actual values for config can be found from existing spinnaker
- Dynamic accounts can be enabled by uncommenting the "dynamicAccount:" comments removing the "{}" as the value


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
