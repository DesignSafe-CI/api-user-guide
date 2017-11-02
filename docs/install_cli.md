---
layout: page
title: Install and Configure the CLI
tagline:
---

One method for interacting with Designsafe as a platform is this 
[Agave command line interface (CLI)](https://agaveapi.co/).
The CLI is a collection of ~150 shell scripts for managing
authorization, files, systems, applications, jobs, and more. 

There are three methods to install the CLI:
1. Run an installer script to install in ~/dscli
2. Manual install to a location of your choosing
3. Pull and use our Docker image

<br>
#### Run an installer script to install in ~/dscli/

Open a terminal window and run the installer script:
```
% curl -L https://raw.githubusercontent.com/DesignSafe-CI/dscli/master/install/install.sh | sh
```

This will install the executables and write an `export PATH` command to your
`~/.bashrc`. To make the changes take effect immediately:
```
% source ~/.bashrc
```

Finally, verify that the CLI has been installed by issuing the following:
```
% ds info

Designsafe-CI API version 1.0.0
TACC Cloud API tenant: designsafe
TACC Cloud API versions:
        Science APIs: 2.2.11
        Reactors API: dev
        TACC Accounting API: v1
```

<br>
#### Manual install to a location of your choosing

Open a terminal window and navigate to your preferred location for installation:

```
% mkdir ds-project && cd ds-project
```

Download and unpack the CLI:
```
% curl -L https://raw.githubusercontent.com/DesignSafe-CI/dscli/master/dscli.tgz -o dscli.tgz
% tar -xvzf dscli.tgz
```

Then add the CLI executables to your `PATH`:
```
% echo "PATH=\$PATH:$PWD/dscli/bin" >> ~/.bashrc
% source ~/.bashrc
```

Finally, verify that the CLI has been installed by issuing the following:
```
% ds info

Designsafe-CI API version 1.0.0
TACC Cloud API tenant: designsafe
TACC Cloud API versions:
        Science APIs: 2.2.11
        Reactors API: dev
        TACC Accounting API: v1
```

<br>
#### Pull a CLI Docker image

The CLI is also available as a Docker image. For this to work, it is assumed you
have a reasonably recent version of [Docker](https://www.docker.com/) installed.
Pull the latest image and verify CLI availability by performing:
```
% docker pull designsafeci/cloud-cli:latest
% docker run -it -v $HOME/.agave:/root/.agave designsafeci/cloud-cli
```

Then from within Docker, verify that the CLI is available:
```
/home$ ds info

Designsafe-CI API version 1.0.0
TACC Cloud API tenant: designsafe
TACC Cloud API versions:
        Science APIs: 2.2.11
        Reactors API: dev
        TACC Accounting API: v1
```

<br>
#### Influential environment variables

<table style="width:100%" border="1px" cellpadding="5">
  <tr>
    <th align="left">environment variable</th>
    <th>options</th>
    <th align="left">description</th>
  </tr>
  <tr>
    <td>AGAVE_JSON_PARSER</td>
    <td align="center">json_mirror, jq, json, python, native</td>
    <td>Set this to your favorite json parser (if installed)</td>
  <tr>
  </tr>
    <td>AGAVE_DISABLE_AUTO_REFRESH</td>
    <td align="center">1, 0</td>
    <td>Set this to `1` to disable auto refresh of the OAuth2 token</td>
  </tr>
</table>

---
Return to the [API Documentation Overview](../index.md)
