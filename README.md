# Nomad incompatibility example

This is attempt to recreate particular issue which hit our servers after upgrading single-node environment to nomad 0.5.6 from nomad 0.5.4.

## Steps to reproduce:
(inside VM, run `vagrant up && vagrant ssh` to get there)

### Start nomad server at version 0.5.4
`vagrant@nomad:~$ sudo /usr/bin/nomad-0.5.4/nomad agent -dev`

### In separate tab, try running job with nomad 0.5.6
```
vagrant@nomad:~$ /usr/bin/nomad-0.5.6/nomad init
Example job file written to example.nomad
vagrant@nomad:~$ /usr/bin/nomad-0.5.6/nomad run example.nomad
Error submitting job: Unexpected response code: 500 (3 error(s) occurred:

* Missing job region
* Job priority must be between [1, 100]
* Task group cache validation failed: 1 error(s) occurred:

* Task redis validation failed: 1 error(s) occurred:

* Missing Log Config)
```

## Why did it happen?
Nomad in new version makes some fields not required in its JSON API, and stops sending them from client.

Validation for old server version complains about invalid JSON, because these fields were previously required.
