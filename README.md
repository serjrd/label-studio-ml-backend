Configs and boilerplates for Label Studio's Machine Learning backend.

## Quickstart

1. Place your scripts for model training & inference inside root directory. Follow the [API guidelines](#api-guidelines) described bellow. You can put everything in a single file, or create 2 separate one say `my_training_module.py` and `my_inference_module.py`
2. Write down your python dependencies in `requirements.txt`
3. Open `wsgi.py` and make your configurations under `init_model_server` arguments:
    ```python
    from my_training_module import training_script
    from my_inference_module import InferenceModel
    init_model_server(
        create_model_func=InferenceModel,
        train_script=training_script,
        ...
    ```
4. Make sure you have docker & docker-compose installed on your system, then run
    ```bash
    docker-compose up
    ```
   
The last command builds and starts ML backend server listening on `http://localhost:9090`. 
Healthchecking running ML backend could be done by:
```bash
$ curl http://localhost:9090/health
{"status":"UP"}
```

Use this URL to initialize new Label Studio project

```bash
label-studio init new_project --ml-backend-url http://localhost:9090
```

## API guidelines


### Inference module
In order to create module for inference, you have to declare the following class:

```python
from htx.base_model import BaseModel

# use BaseModel inheritance provided by pyheartex SDK 
class MyModel(BaseModel):

    def load(self, resources, **kwargs):
        """Here you load the model into the memory. resources is a dict returned by training script"""
        self.model_path = resources["model_path"]
        self.labels = resources["labels"]

    def predict(self, tasks, **kwargs):
        """Here you create list of model results with Label Studio's prediction format, task by task"""
        predictions = []
        for task in tasks:
            # do inference...
            predictions.append(task_prediction)
        return predictions
```

### Training module
Training could be made in a separate environment. The only one convention is that data iterator and working directory are specified as input arguments for training function which outputs JSON-serializable resources consumed later by `load()` function in inference module.

```python
def train(input_iterator, working_dir, **kwargs):
    """Here you gather input examples and output labels and train your model"""
    resources = {"model_path": "some/model/path", "labels": ["aaa", "bbb", "ccc"]}
    return resources
```