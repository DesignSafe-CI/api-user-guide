---
layout: page
title: Modify Existing Systems and Applications
tagline:
---

Agave [systems](create_systems.md) and [apps](create_app.md) are described by
`json` files and added to the Designsafe tenant using the command line interface. To
modify a system or app after it has been added, simply edit the original `json`
file and use the command line interface to push the changes to the Designsafe tenant.

Common examples are shown below.


<br>
#### Modify an Agave system

This is an example Agave storage system `json` file from an
[earlier section](create_systems.md) of this user guide:
```
{
  "id": "jetstream-storage-username",
  "name": "My Jetstream Storage System",
  "description": "My system for storing data on jetstream instance 129.114.104.39",
  "status": "UP",
  "type": "STORAGE",
  "storage": {
    "host": "129.114.104.39",
    "port": 22,
    "protocol": "SFTP",
    "rootDir": "/home/username",
    "homeDir": "/",
    "auth": {
      "publicKey": " <enter public key here> ",
      "privateKey": " <enter private key here> ",
      "username": "username",
      "type": "SSHKEYS"
    }
  }
}
```

This storage system is attached to a [Jetstream instance](https://jetstream-cloud.org/).
Users of this and other cloud environments know that IP addresses for instances may
change. If your Jetstream instance IP address changes, there is no need to delete the old
storage system and create a new one. Instead, you can modify the existing storage system
to use the new IP address.

Edit the original system `json` file to reflect the new IP address (in the case above,
the `description` and `host` fields would need to be edited. Then, perform the 
following:
```
% systems-addupdate -F jetstream-storage-username.json jetstream-storage-username
```

To confirm that the changes have taken place, verbosely list the contents of the
storage system:
```
% systems-list -v jetstream-storage-username
```

<br>
#### Modify an Agave app

This is an example Agave app `json` file from an
[earlier section](create_app_04.md) of this user guide:
```
{
  "name": "fastqc-username",
  "version": "0.11.5",
  "executionType": "HPC",
  "executionSystem": "hpc-tacc-maverick-username",
  "parallelism": "SERIAL",
  "deploymentPath": "/sd2e-apps/fastqc-0.11.5",
  "deploymentSystem": "data-tacc-work-username",
  "defaultProcessorsPerNode": 1,
  "defaultNodeCount": 1,
  "defaultQueue": "normal",
  "label": "FastQC",
  "modules": [ "load tacc-singularity/2.3.1" ],
  "shortDescription": "A Quality Control application for FastQ files",
  "templatePath": "runner-template.sh",
  "testPath": "tester.sh",
  "inputs": [
    {
      "id": "fastq",
      "value": {
        "default": "",
        "visible": true,
        "required": true
      },
      "details": {
        "label": "FASTQ sequence file",
        "showArgument": false
      },
      "semantics": {
        "minCardinality": 1,
        "maxCardinality": 1,
        "ontology": [ "http://edamontology.org/format_1930" ]
      }
    }
  ],
  "parameters": [],
  "outputs": []
}
```

If a new version of FastQC was released (e.g. 0.11.6), that would be cause to 
[create a new app](create_app.md), using this current app as a template. An 
example case where it would be approriate to modify an existing app is,
for example, to a default file for the `fastq` input. 

If you followed the [Create Custom Applications](create_app.md) user guide,
then you may have a sample fastq file located here:

`agave://data-tacc-work-username/sd2e-data/sample/SP1.fq`

Modify the original `app.json` file to include the complete URI to this sample
data as follows (only one line is edited, all other parts untouched):
```
...
  "inputs": [
    {
      "id": "fastq",
      "value": {
        "default": "agave://data-tacc-work-username/sd2e-data/sample/SP1.fq",
        "visible": true,
        "required": true
      },
...
```

Then, update the app in the Designsafe tenant by performing the following:
```
% apps-addupdate -F app.json fastq-username-0.11.5
```

If successful, Agave will automatically append a suffix of `u1`, `u2`, `u3`, etc.,
incrementing each time the app is updated. To confirm that the changes have taken
place, verbosely list the contents of the app:
```
% apps-search name.like='*fastq*'
fastqc-username-0.11.5
fastqc-username-0.11.5u1     # this is the updated app

% apps-list -v fastqc-username-0.11.5u1
```

After carefully testing the updated app, if you are satisfied with its operation,
you may consider disabling the earlier app. The following command toggles the 
`available` flag to `false`:
```
% apps-disable fastqc-username-0.11.5
```


<br>
#### Further help

Additional system and app `json` fields can be found in in the online
[Agave documentation](http://developer.agaveapi.co/).



---
Return to the [API Documentation Overview](../index.md)
