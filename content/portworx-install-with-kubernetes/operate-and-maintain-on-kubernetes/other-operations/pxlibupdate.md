---
title: Update Portworx file system dependencies
keywords: pxlib
description: PX File System Dependency Updates
---

The Portworx Enterprise container ships with an archive containing precompiled modules for the kernels which were available at the time the container was released. If a kernel is released after a Portworx release, a pre-compiled version of the modules will not exist in the container. If Portworx is deployed in a setup which does not have access to mirrors.portworx.com to download the latest modules (i.e. an air-gapped deployment), then Portworx will fail to come up.

Alternatively, in such deployments, you can publish the updated containers to internal container repositories where Portworx can download the new kernel modules.

Regardless of whether your cluster accesses the updated container through mirrors.portworx.com or an internal container registry, you can use that container to update your nodes' Portworx installation to include the latest batch of precompiled modules. Perform the update  by running a DaemonSet in the cluster where Portworx has been installed.

Instructions below will allow you to get a manifest that installs a Daemonset in the cluster for this.

{{<info>}} **Air-gapped clusters**: If your cluster is air gapped, ensure that the following image used by the DaemonSet is present in your internal image registry:

```text
portworx/px-lib:pxfslibs-updater
```
{{</info>}}

If using an internal registry the image should be pulled from the external registry, tagged and push to the internal registry:

```text
docker pull portworx/px-lib:pxfslibs-updater
docker tag portworx/px-lib:pxfslibs-updater <internal registry>/portworx/px-lib:pxfslibs-updater
docker push <internal registry>/portworx/px-lib:pxfslibs-updater
```
In the above command "podman" can be used instead of "docker".

```text
curl -fsL -o pxlibupdate-spec.yaml "https://install.portworx.com?comp=pxlibupdate"
```

If using an internal registry the curl command above, used to generated/download the update spec can be modified to include a parameter to specify the internal registry name.  This will pre-append the internal registry to image name in the spec.  So the URL: "https://install.portworx.com?comp=pxlibupdate&reg=<internal registry>" will set the spec to download the image "<internal registry>/portworx/px-lib:pxfslibs-mver10-update".

Then apply the spec.

```text
kubectl apply -f pxlibupdate-spec.yaml
```

The update is deployed as a daemonset.  Once it is successfully started, a restart of PX will be required, and it will use the updated archive.

To restart PX, label all your Kubernetes nodes as below.

```text
kubectl label nodes --all px/service=restart --overwrite
```

The DaemonSet above will update the pre-compiled module archives within the PX install location on the node (i.e /opt/pwx).

Note that this process assumes that PX is already installed since it will only update an existing installation.  The existing installation's version will be checked for compatibility with the update container and if it's not compatible the update will not be done.
