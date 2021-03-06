---
layout: page
title: Create Custom Applications
tagline: Add your app to the Designsafe tenant
---

At this stage, most of the files that form the Agave app bundle are ready to go.
Your directory structure should be similar to:
```
fastqc/
├── Dockerfile                 # done !
├── build.sh                   # done !
├── deploy.sh
├── example                    # done !
│   ├── SP1.fq                 # done !
│   ├── SP1_fastqc.html        # done ! (can remove)
│   └── SP1_fastqc.zip         # done ! (can remove)
├── fastqc-0.11.5
│   ├── _util                  # done ! (did not change)
│   │   └── container_exec.sh  # done ! (did not change)
│   ├── app.json
│   ├── app.yml
│   ├── job.json
│   ├── runner-template.sh     # done !
│   └── tester.sh              # done !
└── src                        # done !
    ├── README.md              # done !
    └── fastqc_v0.11.5.zip     # done ! (optional)
```
The final step required before you can run the app is to edit the `json` file
descriptions so your app is compatible with Agave.

<br>
#### Write the app description

The Agave app description is contained in `app.json`. All `app.json` files
consist of four essential sections: metadata, input, parameters, and output.
```
{	"metadata":"value",
	"inputs":[],
	"parameters":[],
	"outputs":[]
}
```

The metadata section will contain information about our app name, version, where
and how an instance of the app should run, the location of the app bundle, and
more. There is only one input required: the name (and location) of the fastq file
to be processed. We do not need to specify any parameters or outputs at this time.

A full list of all metdata, inputs, parameters, and outputs conventions can be
found in the [Agave documentation](http://developer.agaveapi.co/#apps).
Rather than try to write the `app.json` from scratch, we will modify the template
to contain the following:
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

The app `name` given is `fastqc-username` (where `username` is your TACC username).
This is a common convention to distinguish your private apps from the publicly 
available apps. It is also a good idea to name an app which is in development
according to this convention. Later, it can be [cloned and published](share_app.md)
to a new public app without the `-username` tag.

Some of the other useful fields are shown above. We use our private execution
system (`hpc-tacc-maverick-username`) to run this app. The `deploymentPath` and
`deploymentSystem` fields show where the app assets (including the wrapper 
script) reside. The `templatePath` and `testPath` fields should match the names
of the files we created. And, as this is a simple `SERIAL`, `HPC` job, it will
be submitted to the `normal` queue using `1` processor

The `inputs` section only contains one input - the `fastq` file to be processed.
Note the `id` of the input paramter, `fastq`, matches the environment variable
in our `runner_template.sh` script. 

The JSON format is unforgiving of typos, so it is a good idea to validate your JSON using
an [online tool](https://jsonlint.com/) to make sure there are no missing 
characters. The YAML file (`app.yml`) is probably not strictly necessary. It is a clone of
the `app.json` file, just in a different format. You may perform the
conversion using [online tools](https://www.json2yaml.com/).


<br>
#### Write a job template

Instructions for running an instance of an app (a job), are located in the 
`job.json` file. Here is an example template for our job:
```
{
  "name": "FastQC-job",
  "appId": "fastqc-username-0.11.5",
  "archive": false,
  "inputs": {
    "fastq": "agave://data-tacc-work-username/sd2e-data/sample/SP1.fq"
  }
}
```

Give the job a unique `name`, the `appId` is a concatenation of the app `name` and
`version` from the `app.json` file above,
and the only input required is a `fastq` file. In this example, we point
the `fastq` variable to our sample data on our private storage system. (We have 
not uploaded that data yet, but we will do it before we submit the job.)

The `archive` flag can be set to `true` or `false`. If `false`, the job is staged
and run in temporary space on the execution system. Job outputs will only be 
downloadable for a limited time (on the order of weeks). If set to `true`, the
new files produced from the job (e.g. outputs, logs) will be copied to a 
system and path of your choosing. Specify the system and path with the `archiveSystem`
and `archivePath` json objects. More on job submission [here](http://developer.agaveapi.co/#job-submission).


<br>
#### Inspect and run the deploy script

The deploy script does two things. It stages your app assets (including the 
wrapper script) on the storage system specified in the app description, and it
creates a new app in the Designsafe tenant using the Agave `apps-addupdate` command.
```
% cd ~/fastqc/
% bash deploy.sh fastqc-0.11.5/
...
Successfully added app fastqc-username-0.11.5
...
```

Among other output, a success message should appear near the bottom, indicating
your app has been successfully deployed to the Designsafe tenant.


<br>
#### Run a test job

Now that your app and assets are staged, the final steps are to stage your sample
data and to run a test job. Create the appropriate directories (if they do not
exist already) and upload the example data:
```
% files-list -S data-tacc-work-username /
...   # contents of your TACC work root directory
% files-mkdir -S data-tacc-work-username -N sd2e-data/sample /
Successfully created folder sd2e-data/sample
% files-upload -S data-tacc-work-username -F example/SP1.fq /sd2e-data/sample/ 
Uploading example/SP1.fq...
######################################################################## 100.0%
```

Make sure the Agave storage system, the path name, and the file name match those
specified in the `job.json` file. When ready, run the test with the `jobs-submit`
command:
```
% cd ~/fastqc/fastqc-0.11.5/
% jobs-submit -F job.json
Successfully submitted job 3305694762687393305-242ac11b-0001-007
```

Track the status of the job with the `jobs-history` command:
```
% jobs-history 3305694762687393305-242ac11b-0001-007
Job accepted and queued for submission.
Attempt 1 to stage job inputs
Identifying input files for staging
Copy in progress
Job inputs staged to execution system
Preparing job for submission.
Attempt 1 to submit job
Fetching app assets from agave://data-tacc-work-username//sd2e-apps/fastqc-0.11.5
Staging runtime assets to agave://hpc-tacc-maverick-username/username/job-3305694762687393305-242ac11b-0001-007-fastqc-job
HPC job successfully placed into normal queue as local job 735953
Job started running
Job completed execution
Job completed. Skipping archiving at user request.

% jobs-list 3305694762687393305-242ac11b-0001-007
3305694762687393305-242ac11b-0001-007 FINISHED
```

*Tip: '`jobs-history -W JOBID`' will follow the progress in real time, similar to '`tail -f`'.*

Once the job has reached `FINISHED` status, you can list the contents of the job
directory, or download and inspect the ouput on your local machine:
```
% jobs-output-list 3305694762687393305-242ac11b-0001-007
% jobs-output-get -r 3305694762687393305-242ac11b-0001-007
```

A successful run will return, among other files, the expected `SP1_fastqc.html`
and `SP1_fastqc.zip` files.

---
Proceed to [Next steps](create_app_05.md)

Go back to [Create Custom Applications](create_app.md)

Return to the [API Documentation Overview](../index.md)
