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
![Authentication](pictures\create-service-principal.png)

### Get the `objectId` using the `clientId` from the previous command:
```
az ad sp show --id ac3638b4-3928-4a75-b4e5-131ec8887d04
```
Screenshot:
![Obtain object id](pictures\get-object-id.png)

### Share the workspace with the created service principal using the command:
```
az ml workspace share \ 
      -w <your-workspace-name> \ 
      -g <your-resurce-group-name> \ 
      --user <objectId> \ 
      --role owner
```
Screenshot: 
![Share workspace](pictures\share-workspaces.png)


### 2. Automated ML Experiment
After we have authenticated we will define an automated ml experiment. For that we need to:

### Register the dataset:
![Dataset registration](pictures\dataset-registered.png)

### Define the ML pipeline:
![Define the ML pipeline](pictures\pipeline-in-azure-ml-studio.png)

### Pipeline structure:
![Pipeline structure](pictures\pipeline-structure.png)

### Pipeline details:
![Pipeline details](pictures\pipeline-overview.png)

### Published pipeline:
![Published the pipeline](pictures\published-pipeline.png)

### Run the pipeline:
![Run the pipeline](pictures\completed-run.png)

### Get the best model:
![Get the best model](pictures\completed-model.png)


### 3. Deploy the best model

### Go to the model tab:
![Get the best model](pictures\best-model.png)

### Click the deploy button which will give us a published endpoint:
![Get the best model](pictures\deployed-model.png)

### 4. Enable logging
It is done with just a small python script:

```python
from azureml.core import Workspace
from azureml.core.webservice import Webservice

# Requires the config to be downloaded first to the current working directory
ws = Workspace.from_config()

# Set with the deployment name
name = "bank-marketing-best-model"

# load existing web service
service = Webservice(name=name, workspace=ws)
# enable application insight
service.update(enable_app_insights=True)
```
### And we can se the logs from application insights:
![logs](pictures\logs-from-ai.png)

### 5. Swagger documentation

The nice thing about ml studio is that the published endpoint comes with the swagger.json which is a way to document APIs that is highly human and machine readable. 

### Using this file we can consume the documentation via swagger ui:
![swagger request](pictures\swagger-request.png)

### and the response:
![swagger response](pictures\swagger-request.png)

### 6. Consume model endpoints
We can do it with a small python script:
```python
import requests
import json

# URL for the web service, should be similar to:
# 'http://8530a665-66f3-49c8-a953-b82a2d312917.eastus.azurecontainer.io/score'
scoring_uri = 'http://f3734569-4678-436c-bc6c-1b65c44a21cf.westeurope.azurecontainer.io/score'
# If the service is authenticated, set the key or token
key = 'AWgj8ZdjCdqjgbDCiI1Czh6eaEhEJOGE'

# Two sets of data to score, so we get two results back
data = {"data":
        [
          {
            "age": 17,
            "campaign": 1,
            "cons.conf.idx": -46.2,
            "cons.price.idx": 92.893,
            "contact": "cellular",
            "day_of_week": "mon",
            "default": "no",
            "duration": 971,
            "education": "university.degree",
            "emp.var.rate": -1.8,
            "euribor3m": 1.299,
            "housing": "yes",
            "job": "blue-collar",
            "loan": "yes",
            "marital": "married",
            "month": "may",
            "nr.employed": 5099.1,
            "pdays": 999,
            "poutcome": "failure",
            "previous": 1
          },
          {
            "age": 87,
            "campaign": 1,
            "cons.conf.idx": -46.2,
            "cons.price.idx": 92.893,
            "contact": "cellular",
            "day_of_week": "mon",
            "default": "no",
            "duration": 471,
            "education": "university.degree",
            "emp.var.rate": -1.8,
            "euribor3m": 1.299,
            "housing": "yes",
            "job": "blue-collar",
            "loan": "yes",
            "marital": "married",
            "month": "may",
            "nr.employed": 5099.1,
            "pdays": 999,
            "poutcome": "failure",
            "previous": 1
          },
      ]
    }
# Convert to JSON string
input_data = json.dumps(data)
with open("data.json", "w") as _f:
    _f.write(input_data)

# Set the content type
headers = {'Content-Type': 'application/json'}
# If authentication is enabled, set the authorization header
headers['Authorization'] = f'Bearer {key}'

# Make the request and display the response
resp = requests.post(scoring_uri, input_data, headers=headers)
print(resp.json())
```

## Screen Recording
You can find a screen cast here: https://youtu.be/cCXFPUZrlMg