# Operationalizing Machine Learning with Azure ML

This project has the goal of demonstrating the end to end workflow of operationalizing a machine learning pipeline using azure ml studio.

## Architectural Diagram

 ![Architecture](/pictures/Architecture-diagram.png)

## Key Steps
The key steps of this project are defined below:

### 1. Authentication
In order to work with the azure ml cli it is necessary to login to the azure accout and create a service principal that would have an owner role in the azure ml workspace.

### Add a new service principal named `ml-auth` with the following command:
```
az ad sp create-for-rbac --sdk-auth --name ml-auth
```
Screenshot:
![Authentication](/pictures/create-service-principal.png)

### Get the `objectId` using the `clientId` from the previous command:
```
az ad sp show --id ac3638b4-3928-4a75-b4e5-131ec8887d04
```
Screenshot:
![Obtain object id](/pictures/get-object-id.png)

### Share the workspace with the created service principal using the command:
```
az ml workspace share \ 
      -w <your-workspace-name> \ 
      -g <your-resurce-group-name> \ 
      --user <objectId> \ 
      --role owner
```
Screenshot: 
![Share workspace](/pictures/share-workspaces.png)


### 2. Automated ML Experiment
After we have authenticated we will define an automated ml experiment. For that we need to:

### Register the dataset
In order to have access to the data inside the azure ml studio a dataset must be registered. It is done in the dataset section of the ml studio. Azure ml studio offers different possibilities to register a dataset - from local files, local datastores, web files and even from open datasets. We will use From web files option: 

![Dataset registration](/pictures/dataset-registered.png)

### Define the ML pipeline
For the purpose of this project the training pipeline will be fairly simple - it will contain only the dataset and the automl steps:

![Define the ML pipeline](/pictures/pipeline-in-azure-ml-studio.png)

### Pipeline structure:
![Pipeline structure](/pictures/pipeline-structure.png)

### Pipeline details:
![Pipeline details](/pictures/pipeline-overview.png)

### Published pipeline
To be able to trigger the pipeline from other CI/CD pipelines we need to have the pipeline published:

![Published the pipeline](/pictures/published-pipeline.png)

### Run the pipeline
We will run the pipeline to generate the ml model:

![Run the pipeline](/pictures/completed-run.png)

### Get the best model:
The automl pipeline in particular does a number of runs to determine the best model architecture for us, all we have to do is to select the best run for further deployment:

![Get the best model](/pictures/completed-model.png)

### 3. Deploy the best model
This is the step that makes our model useful outside the studio and one step closer to the users.

To deploy a model from a run we need to go to the model tab:
![Get the best model](/pictures/best-model.png)

Click the deploy button which will give us a published endpoint:
![Get the best model](/pictures/deployed-model.png)

### 4. Enable logging
This step is a crucial step for any webservice that is intended for production. Logging offers insights and early warning signs that our service migh not be doing well. Logging is used as well for investigation of any incidents or failures.

Running the [enable-ai.py](enable-ai.py) script will enable logging for our model endpoint.

### And we can se the logs from application insights:
![logs](/pictures/logs-from-ai.png)

### 5. Swagger documentation

The nice thing about ml studio is that the published endpoint comes with the swagger.json which is a way to document APIs that is highly human and machine readable. 

### Using this file we can consume the documentation via swagger ui:
![swagger request](/pictures/swagger-request.png)

### and the response:
![swagger response](/pictures/swagger-request.png)

### 6. Consume model endpoints
Check [endpoint.py](endpoint.py) script to see an example of consuming the REST endpoint.

## Screen Recording
You can find a screen cast here: https://youtu.be/cCXFPUZrlMg

## Further Improvements
In order to make this project ready for prime time I would investigate and add model versioning. This is required to allow for fallback in case the new model is faulty. I would add as well a benchmarking step to the pipeline to be able to asses the quality of the new model.