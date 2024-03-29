# Virtual Machines in PEPR

[KubeVirt](https://github.com/kubevirt) is a CNCF incubating project to integrate runtime virtualization with Kubernetes. KubeVirt is the leading(only) solution ready for supporting Virtual Machines in K8s.

## Ideal Future State

Virtual Machine orchestration has much to be desired and falls short compared with running containerized workloads.

Questions that need to be answered for a great, enterprise solution.
1. How can VMs be packaged and moved around like a container registry?

2. VMWare and similar solutions, provide Orchestration UIs. (create a VM, snapshots, backups, move, etc...). While KubeVirt API provides a mechanism to complete this work, it is low level and requires an SRE to be familiar with Kubernetes AND KubeVirt to perform relatively simple tasks. A VM orchestration solution is likely required.

Ideally,

1. VMs could be integrated by an engineer in a Zarf package and included in the zarf artifact when running `zarf package create`
2. Upon a  `zarf package deploy` the VM would be uploaded and created
3. A VM orchestration, either automatically or via User action, solution would start the VM
4. A user could access a VNC(remote) session connection using Baffles to the running VM, as necessary.


## Virtual Machine 

These sections outline the required steps to achieve a running virtual machine from nothing. These steps are currently performed manually by an SRE, as there is not automation or orchestration built.

Prerequisite: [KubeVirt Big Bang Package](https://repo1.dso.mil/platform-one/big-bang/apps/third-party/kubevirt) installed and running.

### Uploading
KubeVirt supports a wide range of virtual machine image types.  First this image needs to be made available to the operator. For this, SREs can leverage Containerized Data Importer [CDI](https://github.com/kubevirt/containerized-data-importer) (and [CDI Big Bang Package](https://repo1.dso.mil/platform-one/big-bang/apps/sandbox/cdi) )to upload the image to a Persistent Volume.

The most immediate mechanism to upload is interacting the the CDI API via `virtctl image-upload ...`

#### Options when using Zarf

The options are:

raw or qcow2 formatted images, via HTTP/HTTPS
containerDisk images from a docker registry (which internally contain a raw or qcow2 image, but inside a real Docker image, with the file on a Docker layer)
The containerDisk can be pulled by CDI itself (in which case it needs image credentials) or CDI can have the node pull the image pullMethod: node. When the node pulls the image, we did test that it can come from a Zarf registry, the Zarf image rewrite works, and Zarf imagePullSecret credentials work.

This basically means that you can avoid a real "upload" step, and it can just be part of normal zarf Docker image mirroring.

### Install

Virtual Machines are created using the `VirtualMachine` custom resource.

```yaml
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: testvm
spec:
  running: false
  template:
    metadata:
      labels:
        ...
    spec:
      domain:
        devices:
          disks:
            ...
          interfaces:
          ...
        resources:
          ...
      networks:
      ...
      volumes:
        - name: containerdisk
          containerDisk:
            image: quay.io/kubevirt/cirros-container-disk-demo
        - name: cloudinitdisk
          ...
```
*[full example](https://kubevirt.io/labs/manifests/vm.yaml)*

Note: this is a lightweight container example, but KubeVirt supports "real" VMs with larger disks and resource requirements see [SECTION TBD]()

### Running
Once a Virtual Machine (vm) custom resource is created. An SRE may change the `spec.running` field to true in the definition or more preferably use the `virtctl start testvm` cli utility.

When the Virtual Machine is running, users may access the VM using VNC. 

### Example

Excellent [video](https://drive.google.com/file/d/1xeaFuAOo1A-Zyr1SBc1RTOxVu4SZ9NPa/view) showing a VM running from a zarf deployment with Istio and the associated [naps-dev/mockingbird](https://github.com/naps-dev/mockingbird) github repository. 

## Baffles

[Baffles](https://baffles.dev) is an open source tool built by Defense Unicorns and intends to be a delightful experience allowing the end user to focus on their mission/task. As it relates to VMs, Baffles integrates with Kubevirt API to connect to VNC sessions.

In order to "be discoverable" the VM would need a `BafflesApplication` custom resource

```yaml
apiVersion: app.baffles.dev/v1
kind: BafflesApplication
metadata:
  name: widnows10
  namespace: default
spec:
  displayName: "Windows 10"
  url: "ws://<NODE>:<PORT>/k8s/apis/subresources.kubevirt.io/v1alpha3/namespaces/default/virtualmachineinstances/iso-win10/vnc"
  category: "vm"
```

# PEPR 

Components required to integrate a Virtual Machine with Big Bang:

1. VM gets built as a compatible Docker Image ([example](https://github.com/naps-dev/mockingbird/blob/main/Dockerfile))
2. Manifests/Helm Chart
   1. Virtal Machine KubeVirt CR, note ports and selector labels ([example](https://github.com/naps-dev/mockingbird/blob/main/chart/templates/virtualmachine.yaml))
   2. Kubernetes Service, with ports specified ([example](https://github.com/naps-dev/mockingbird/blob/main/chart/templates/service.yaml))
   3. Istio Virtual Service, with ports specified ([example](https://github.com/naps-dev/mockingbird/blob/main/chart/templates/virtualservice.yaml))
3. (If Baffles) A Baffles Application CR created 

Not yet identified/to be further investigated:
1. Logging/Monitoring, Kubevirt creates a "virt-handler" pod that can be integrated with logging/monitoring. However, the operations of a VM are not integrated out of the box. 
2. Iron Bank scanning of Virtual Machine docker built images 
3. Database/Object Storage - will depend on the target environment and needs of the VM. 
4. SSO - will depend on the virtual machine, needs to be investigated further