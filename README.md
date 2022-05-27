# fuse-on-openshift

## Overview
This uses the [smarter device manager](https://gitlab.com/arm-research/smarter/smarter-device-manager) to talk to the k8s plugin api 
and register the node's /dev/fuse device as something that can be requested as a limit during pod creation.

## Configuration
The main configuration points are `conf.yml` and `smarter-device-manager.yml`. In conf.yml you can add other devices that can be registered.
This project is set up for just fuse but you could add ttys, video cards, sound cards, or any other local hardware.

The second configuration point is in the node selector of [`smarter-device-manager.yml`](smarter-device-manager.yml#L22). By default you can annotate nodes
with `smarter-device-manager : enabled` but you could also just change the selector to match whatever your cluster needs.

## Installation
Create the target namespace and apply the kustomize content. You will need to have cluster-admin privileges.
```bash
[]$ oc new-project smarter-devices
Now using project "smarter-devices" on server "..."
# ... skip ...
[]$ oc apply -k smarter-devices
# ... skip ...
[]$ oc adm policy add-scc-to-user smarter-device-scc -z smarter-device
```

## Use
Any pod that needs to use a device needs to have a resource limit like the following:
```
resources:
  limits:
    'smarter-devices/fuse': 1
```
The given device path `/dev/fuse` is translated to `smarter-devices/fuse` and limits the pod to being deployed on nodes that have 
the DaemonSet exporting the configured device.

