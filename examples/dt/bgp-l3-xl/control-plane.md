# Configuring networking and deploy the OpenStack control plane

## Assumptions

- A storage class called `local-storage` should already exist.
- An infrastructure of spine/leaf routers exists, is properly connected to the
  OCP nodes and the routers are configured to support BGP.

## Initialize

Switch to the "openstack" namespace
```
oc project openstack
```
Change to the bgp-l3-xl/control-plane directory
```
cd architecture/examples/dt/bgp-l3-xl/control-plane
```
Edit the [networking/nncp/values.yaml](control-plane/networking/nncp/values.yaml) and
[service-values.yaml](control-plane/service-values.yaml) files to suit
your environment.
```
vi networking/nncp/values.yaml
vi service-values.yaml
```

## Apply node network configuration

Generate the node network configuration
```
kustomize build networking/nncp > nncp.yaml
```
Apply the NNCP CRs
```
oc apply -f nncp.yaml
```
Wait for NNCPs to be available
```
oc wait nncp -l osp/nncm-config-type=standard --for jsonpath='{.status.conditions[0].reason}'=SuccessfullyConfigured --timeout=300s
```

## Apply the remaining networking configuration

Generate the networking CRs.
```
kustomize build networking > networking.yaml
```
Apply the CRs
```
oc apply -f networking.yaml
```

## Apply control-plane configuration

Generate the control-plane CRs.
```
kustomize build > control-plane.yaml
```
Apply the CRs
```
oc apply -f control-plane.yaml
```

Wait for control plane to be available
```
oc wait osctlplane controlplane --for condition=Ready --timeout=600s
```

