# cOS + k3s + fleet = :heart:

This repository is a sample of a custom cOS derivative which updates are managed by fleet.


## 1) Build the OS image

```bash
# cd os
# docker build -t myimage .
```

## 2) Push the docker image


```bash
# docker push myimage
```

## 3) Prepare a cOS VM

Download an ISO, or a qcow image from the Github artifacts of cOS. Or generate an iso of the image (check [here](https://github.com/mudler/os2) for another example). 

If deploying on AWS/openstack/Cloud, use the `fleet-cloud-init.yaml` file as userdata. If deploying on baremetal/VMs, place `fleet-cloud-init.yaml` in `/oem` after install (or run the installer with `cos-installer $DEVICE --config fleet-cloud-init.yaml`).

Reboot, after some bootstraping time, you should have a k3s cluster with fleet and [system-upgrade-controller](https://github.com/rancher/system-upgrade-controller) deployed. 

## 4) Upgrade with fleet

Add your fleet repository to the fleet cluster:

```bash
cat > example.yaml << "EOF"
apiVersion: fleet.cattle.io/v1alpha1
kind: GitRepo
metadata:
  name: upgrade
  # This namespace is special and auto-wired to deploy to the local cluster
  namespace: fleet-local
spec:
  # Everything from this repo will be ran in this cluster. You trust me right?
  repo: "https://github.com/mudler/fleet-sample"
  branch: "main"
EOF

kubectl apply -f example.yaml
```

An example of how to trigger an upgrade with fleet is in `manifests/upgrade.yaml`. Edit the image with the one generated in the previous steps, and commit it to your **fleet repository**, At this point you should see the upgrade job to kick-in, the system will reboot afterwards.

