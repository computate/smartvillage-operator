
# Prerequisites

## Install kubernetes.core Ansible Galaxy Collection

```bash
pip3 install kubernetes jmespath
ansible-galaxy collection install kubernetes.core
```

### Upgrade kubernetes.core Ansible Galaxy Collection if necessary

```bash
ansible-galaxy collection install kubernetes.core -U
```

# How the operator was initialized

```bash
operator-sdk init --plugins=ansible --domain computate.org
```

# Build and deploy the operator to an OpenShift environment
```bash
oc login ...
make docker-build docker-push deploy
```

## Initialize TrafficFlowObserved model

```bash
operator-sdk create api --group smartvillage --version v1 --kind TrafficFlowObserved --generate-role
ansible-playbook write-smart-data-model-templates.yaml -e ENTITY_TYPE=TrafficFlowObserved
```

## Initialize CrowdFlowObserved model

```bash
operator-sdk create api --group smartvillage --version v1 --kind CrowdFlowObserved --generate-role
ansible-playbook write-smart-data-model-templates.yaml -e ENTITY_TYPE=CrowdFlowObserved
```

## Initialize SmartTrafficLight model

```bash
operator-sdk create api --group smartvillage --version v1 --kind SmartTrafficLight --generate-role
ansible-playbook write-smart-data-model-templates.yaml -e ENTITY_TYPE=SmartTrafficLight
```

## Initialize OrionLDContextBroker model

```bash
operator-sdk create api --group smartvillage --version v1 --kind OrionLDContextBroker --generate-role
ansible-playbook write-smart-data-model-templates.yaml -e ENTITY_TYPE=OrionLDContextBroker
```

## Initialize IoTAgentJson model

```bash
operator-sdk create api --group smartvillage --version v1 --kind IoTAgentJson --generate-role
ansible-playbook write-smart-data-model-templates.yaml -e ENTITY_TYPE=IoTAgentJson
```

## Initialize EdgeAmqBroker model

```bash
operator-sdk create api --group smartvillage --version v1 --kind EdgeAmqBroker --generate-role
ansible-playbook write-smart-data-model-templates.yaml -e ENTITY_TYPE=EdgeAmqBroker
```

## Initialize SmartaByarSmartVillage model

```bash
operator-sdk create api --group smartvillage --version v1 --kind SmartaByarSmartVillage --generate-role
ansible-playbook write-smart-data-model-templates.yaml -e ENTITY_TYPE=SmartaByarSmartVillage
```

Create a keycloak-client-secret-smartvillage secret on OpenShift

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: keycloak-client-secret-smartvillage
stringData:
  CLIENT_ID: smartvillage
  CLIENT_SECRET: ...
type: Opaque
```

# Install AMQ Broker on microshift

Follow the instructions in the docs [Deploying AMQ Broker on OpenShift Container Platform using the AMQ Broker Operator](https://access.redhat.com/documentation/en-us/red_hat_amq_broker/7.11/html/deploying_amq_broker_on_openshift/deploying-broker-on-ocp-using-operator_broker-ocp)


```bash
cd ~/Downloads/amq-broker-operator-7.11.0-ocp-install-examples/deploy/
oc create namespace smartvillage
oc config set-context --current --namespace=smartvillage
oc apply -f service_account.yaml
oc apply -f role.yaml
oc apply -f role_binding.yaml
oc apply -f election_role.yaml
oc apply -f election_role_binding.yaml
oc apply -f crds/broker_activemqartemis_crd.yaml
oc apply -f crds/broker_activemqartemisaddress_crd.yaml
oc apply -f crds/broker_activemqartemisscaledown_crd.yaml
oc apply -f crds/broker_activemqartemissecurity_crd.yaml
oc apply -f operator.yaml
```

# Install kubectl command

https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

# Install the Smart Village Operator on microshift

```bash
mkdir ~/.local/src
git clone git@github.com:computate-org/smartvillage-operator.git ~/.local/src/smartvillage-operator
git clone git@github.com:computate-org/smartabyar-smartvillage.git ~/.local/src/smartabyar-smartvillage
cd ~/.local/src/smartvillage-operator
make deploy
cd ~/.local/src/smartabyar-smartvillage
oc apply -k openshift/kustomize/overlays/microshift/
```

# OpenShift Local Deployment

- Install the Edge AMQ Broker for MQTT and AMQP messaging. 

```bash
oc apply -k ~/.local/src/smartabyar-smartvillage/openshift/kustomize/overlays/local/edgeamqbrokers/
# Finishes in about 32 seconds
```

- Install the Orion-LD Context Broker and MongoDB, to receive device entity data and activate NGSI-LD APIs. 

```bash
oc apply -k ~/.local/src/smartabyar-smartvillage/openshift/kustomize/overlays/local/orionldcontextbrokers/
# Finishes in about 51 seconds
```

- Install the IoT Agent JSON, to receive MQTT or AMQP device data as JSON. 

```bash
oc apply -k ~/.local/src/smartabyar-smartvillage/openshift/kustomize/overlays/local/iotagentjsons/
# Finishes in about 15 seconds
```

- Install the Smarta Byar Smart Village platform, to receive context broker subscription data and enable data science APIs. 

```bash
oc apply -k ~/.local/src/smartabyar-smartvillage/openshift/kustomize/overlays/local/smartabyarsmartvillages/
# Finishes in about 240 seconds
```

- Install a TrafficFlowObserved Smart Data Model entity, to integrate it with the Iot Agent, Context Broker, and Smart Village application. 

```bash
oc apply -k ~/.local/src/smartabyar-smartvillage/openshift/kustomize/overlays/local/trafficflowobserveds/
# Finishes in about 15 seconds
```

