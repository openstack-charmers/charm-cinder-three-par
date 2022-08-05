# This repository has moved 

This repository has moved to https://opendev.org/openstack/charm-cinder-three-par.


# charm-cinder-three-par

# Overview


Cinder is the OpenStack block storage (volume) service and allow for different
backends to be used to provision volumes. The cinder 3PAR charm provides integration
between Cinder service and HPE 3PAR storage array solution. Users can request volumes
using OpenStack APIs and get them provisioned on 3PAR and distributed with either
Fiber Channel or iSCSI connection.

# Usage

## Configuration

This section covers common and/or important configuration options. See file
`config.yaml` for the full list of options, along with their descriptions and
default values. See the [Juju documentation][juju-docs-config-apps] for details
on configuring applications.

## Deployment

### Pre-deployment Setup with Fiber Channel

HPE 3PAR Fiber Channel demands cinder-volume service to be run on baremetal with direct
access to the fiber channel interfaces.

In this type of deployment, break cinder into two applications:

```yaml
    cinder-api:
      charm: cs:cinder
      options:
        enabled-services: "api,scheduler"
        ...
    cinder-volume:
      charm: cs:cinder
      options:
        enabled-services: "volume"
```

Cinder-api can be mapped to lxc containers while cinder-volume needs to be placed on 
a host with access to the fiber channel backend.

### HPE3PAR-backed storage

Cinder can be backed by HPE 3PAR SAN Array, which provides commercial hardware backend
for the volumes.

File `cinder.yaml` contains the following:

```yaml
    cinder-three-par:
      driver-type: fc
      san-ip: 1.2.3.4
      san-login: CHANGE_TO_LOGIN
      san-password: CHANGE_TO_PWD
      hpe3par-username: CHANGE_TO_USERNAME
      hpe3par-password: CHANGE_TO_PWD
      hpe3par-api-url: https://<API_IP>/
      hpe3par-cpg: <SELECT A CPG>
      hpe3par_cpg_snap: <SELECT A CPG>
```

Here, Cinder HPE 3PAR backend is deployed to a container on machine '1' 
and related the cinder subordinate charm:

    juju deploy --to lxd:1 --config cinder-three-par.yaml cinder
    juju deploy cinder-three-par
    juju add-relation cinder-three-par:storage-backend cinder:storage-backend
    
Optionally, set option:
```
      hpe3par-debug: True
```

To gather more logs on the deployment.

# Developing

Create and activate a virtualenv with the development requirements:

    virtualenv -p python3 venv
    source venv/bin/activate
    pip install -r requirements-dev.txt

# Testing

The Python operator framework includes a very nice harness for testing
operator behaviour without full deployment. You can run tests with tox.
