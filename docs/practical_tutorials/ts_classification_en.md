[简体中文](ts_classification.md) | English

# PaddleX 3.0 Time Series Classification Pipeline — Heartbeat Monitoring Time Series Classification Tutorial

PaddleX offers a rich set of pipelines, each consisting of one or more models tailored to solve specific scenario-based tasks. All PaddleX pipelines support quick trials, and if the results do not meet expectations, they also allow fine-tuning with private data. PaddleX provides Python APIs for easy integration into personal projects. Before use, you need to install PaddleX. For installation instructions, refer to [PaddleX Local Installation Guide](../installation/installation_en.md). This tutorial introduces the usage of the pipeline tool with an example of heartbeat time series data classification.

## 1. Select a Pipeline
First, choose the corresponding PaddleX pipeline based on your task scenario. For this task, the goal is to train a time series classification model based on heartbeat monitoring data to classify heartbeat time series conditions. Recognizing this as a time series classification task, we select PaddleX's Time Series Classification Pipeline. If unsure about the task-pipeline correspondence, consult the [PaddleX Pipeline List (CPU/GPU)](../support_list/models_list_en.md) for pipeline capabilities.

## 2. Quick Experience
PaddleX offers two ways to experience its capabilities: locally or on the **Baidu AIStudio Community**.

* Local Experience:
```python
from paddlex import create_model
model = create_model("TimesNet_cls")
output = model.predict("https://paddle-model-ecology.bj.bcebos.com/paddlex/ts/demo_ts/ts_cls.csv", batch_size=1)
for res in output:
    res.print(json_format=False)
    res.save_to_csv("./output/")
```
* AIStudio Community Experience: Access the [Official Time Series Classification Application](https://aistudio.baidu.com/community/app/105707/webUI?source=appCenter) to experience time series classification capabilities.

Note: Due to the tight coupling between time series data and scenarios, the online experience of official models is tailored to a specific scenario and not a universal solution. It does not support arbitrary files for model effect evaluation. However, after training a model with your scenario data, you can select your trained model and use corresponding scenario data for online experience.

## 3. Choose a Model
PaddleX provides a time series classification model. Refer to the [Model List](../support_list/models_list_en.md) for details. The model benchmark is as follows:

| Model Name | Acc (%) | Model Size (M) | Description |
|-|-|-|-|
| TimesNet_cls | 87.5 | 792K | TimesNet is an adaptive and high-precision time series classification model through multi-cycle analysis |

**Note: The evaluation set for the above accuracy metrics is UWaveGestureLibrary.**

## 4. Data Preparation and Verification
### 4.1 Data Preparation
To demonstrate the entire time series classification process, we will use the public [Heartbeat Dataset](https://paddle-model-ecology.bj.bcebos.com/paddlex/data/ts_classify_examples.tar) for model training and validation. The Heartbeat Dataset is part of the UEA Time Series Classification Archive, addressing the practical task of heartbeat monitoring for medical diagnosis. The dataset comprises multiple time series groups, with each data point consisting of a label variable, group ID, and 61 feature variables. This dataset is commonly used to test and validate the performance of time series classification prediction models.

We have converted the dataset into a standard format, which can be obtained using the following commands. For data format details, refer to the [Time Series Classification Module Development Tutorial](../module_usage/tutorials/ts_modules/time_series_classification_en.md).

Dataset Acquisition Command:

```bash
cd /path/to/paddlex
wget https://paddle-model-ecology.bj.bcebos.com/paddlex/data/ts_classify_examples.tar -P ./dataset
tar -xf ./dataset/ts_classify_examples.tar -C ./dataset/
```

**Data Considerations**
  * Based on collected real data, clarify the classification objectives of the time series data and define corresponding classification labels. For example, in stock price classification, labels might be "Rise" or "Fall." For a time series that is "Rising" over a period, it can be considered a sample (group), where each time point in this period shares a common group_id.
  * Uniform Time Series Length: Ensure that the length of the time series for each group is consistent.
Missing Value Handling: To guarantee the quality and integrity of the data, missing values can be imputed based on expert experience or statistical methods.
  * Non-Duplication: Ensure that the data is collected row by row in chronological order, with no duplication of the same time point.


### 4.2 Data Validation
Data Validation can be completed with just one command:

```
python main.py -c paddlex/configs/ts_classification/TimesNet_cls.yaml \
    -o Global.mode=check_dataset \
    -o Global.dataset_dir=./dataset/ts_classify_examples
```
After executing the above command, PaddleX will validate the dataset, summarize its basic information, and print `Check dataset passed !` in the log if the command runs successfully. The validation result file is saved in `./output/check_dataset_result.json`, and related outputs are saved in the current directory's `./output/check_dataset` directory, including example time series data and class distribution histograms.

```bash
{
  "done_flag": true,
  "check_pass": true,
  "attributes": {
    "train_samples": 82620,
    "train_table": [
      [ ...
    ],
    ],
    "val_samples": 83025,
    "val_table": [
      [ ...
      ]
    ]
  },
  "analysis": {
    "histogram": "check_dataset/histogram.png"
  },
  "dataset_path": "./dataset/ts_classify_examples",
  "show_type": "csv",
  "dataset_type": "TSCLSDataset"
}
```
The above verification results have omitted some data parts. `check_pass` being True indicates that the dataset format meets the requirements. Explanations for other indicators are as follows:

* `attributes.train_samples`: The number of samples in the training set of this dataset is 58317.
* `attributes.val_samples`: The number of samples in the validation set of this dataset is 73729.
* `attributes.train_table`: Sample data rows from the training set of this dataset.
* `attributes.val_table`: Sample data rows from the validation set of this dataset.

**Note**: Only data that passes the verification can be used for training and evaluation.

### 4.3 Dataset Format Conversion / Dataset Splitting (Optional)
If you need to convert the dataset format or re-split the dataset, please refer to Section 4.1.3 in the [Time Series Classification Module Development Tutorial](../module_usage/tutorials/ts_modules/time_series_classification_en.md).

## 5. Model Training and Evaluation

### 5.1 Model Training
Before training, ensure that you have validated the dataset. To complete PaddleX model training, simply use the following command:

```bash
python main.py -c paddlex/configs/ts_classification/TimesNet_cls.yaml \
-o Global.mode=train \
-o Global.dataset_dir=./dataset/ts_classify_examples \
-o Train.epochs_iters=5 \
-o Train.batch_size=16 \
-o Train.learning_rate=0.0001 \
-o Train.time_col=time \
-o Train.target_cols=dim_0,dim_1,dim_2 \
-o Train.freq=1 \
-o Train.group_id=group_id \
-o Train.static_cov_cols=label
```
PaddleX supports modifying training hyperparameters and single-machine single-GPU training (time-series models only support single-GPU training). Simply modify the configuration file or append command-line parameters.

Each model in PaddleX provides a configuration file for model development to set relevant parameters. Model training-related parameters can be set by modifying the `Train` fields in the configuration file. Some example parameter descriptions in the configuration file are as follows:

* `Global`:
  * `mode`: Mode, supports dataset validation (`check_dataset`), model training (`train`), model evaluation (`evaluate`), and single-case testing (`predict`);
  * `device`: Training device, options include `cpu`, `gpu`. Check the [Model Support List](../support_list/models_list_en.md) for models supported on different devices.
* `Train`: Training hyperparameter settings;
  * `epochs_iters`: Number of training epochs;
  * `learning_rate`: Training learning rate;
  * `batch_size`: Training batch size per GPU;
  * `time_col`: Time column, set the column name of the time series dataset's time column based on your data;
  * `target_cols`: Set the column names of the target variables of the time series dataset based on your data. Can be multiple, separated by commas. Generally, the more relevant the target variables are to the prediction target, the higher the model accuracy. In this tutorial, the heartbeat monitoring dataset has 61 feature variables for the time column, such as `dim_0, dim_1`, etc.;
  * `freq`: Frequency, set the time frequency based on your data, e.g., 1min, 5min, 1h;
  * `group_id`: A group ID represents a time series sample, and time series sequences with the same ID form a sample. Set the column name of the specified group ID based on your data, e.g., `group_id`.
  * `static_cov_cols`: Represents the category ID column of the time series. The same sample has the same label. Set the column name of the category based on your data, e.g., `label`.
For more hyperparameter introductions, please refer to [PaddleX Time Series Task Model Configuration File Parameter Description](../module_usage/instructions/config_parameters_time_series_en.md).

**Note**:

* The above parameters can be set by appending command-line parameters, e.g., specifying the mode as model training: `-o Global.mode=train`; specifying the first GPU for training: `-o Global.device=gpu:0`; setting the number of training epochs to 10: `-o Train.epochs_iters=10`.
* During model training, PaddleX automatically saves model weight files, with the default being `output`. To specify a save path, use the `-o Global.output` field in the configuration file.

<details>
   <summary> More Details (Click to Expand)  </summary>

* During the model training process, PaddleX automatically saves the model weight files, with the default directory being output. If you need to specify a different save path, you can configure it through the `-o Global.output` field in the configuration file.
* PaddleX abstracts away the concepts of dynamic graph weights and static graph weights from you. During model training, both dynamic and static graph weights are produced simultaneously. By default, static graph weights are selected for inference.
* When training other models, you need to specify the corresponding configuration file. The correspondence between models and configuration files can be found in the [PaddleX Model List (CPU/GPU)](../support_list/models_list_en.md)

After completing the model training, all outputs are saved in the specified output directory (default is `./output/`), typically including the following:

**Explanation of Training Outputs:**

After completing the model training, all outputs are saved in the specified output directory (default is `./output/`), typically including the following:

`train_result.json`: A training result record file that logs whether the training task was completed normally, as well as the output weight metrics, relevant file paths, etc.
`train.log`: A training log file that records changes in model metrics, loss values, and other information during the training process.
`config.yaml`: The training configuration file that records the hyperparameter configurations for this training session.
`best_accuracy.pdparams.tar`, `scaler.pkl`, `.checkpoints`, `.inference*`: Files related to model weights, including network parameters, optimizers, static graph network parameters, etc.

</details>

### 5.2 Model Evaluation
After completing model training, you can evaluate the specified model weights file on the validation set to verify the model's accuracy. Using PaddleX for model evaluation requires just one command:

```
    python main.py -c paddlex/configs/ts_classification/TimesNet_cls.yaml \
    -o Global.mode=evaluate \
    -o Global.dataset_dir=./dataset/ts_classify_examples \
    -o Evaluate.weight_path=./output/best_model/model.pdparams
```

Similar to model training, model evaluation supports setting through modifying the configuration file or appending command-line parameters.

**Note**: When evaluating the model, you need to specify the model weights file path. Each configuration file has a default weight save path built-in. If you need to change it, simply set it by appending a command-line parameter, such as `-o Evaluate.weight_path=./output/best_model/model.pdparams`.

After completing the model evaluation, typically, the following outputs are generated:

Upon completion of model evaluation, `evaluate_result.json` will be produced, which records the evaluation results, specifically, whether the evaluation task was completed successfully, and the model's evaluation metrics, including Acc Top1.


### 5.3 Model Optimization
By adjusting the number of training epochs appropriately, you can control the depth of model training to avoid overfitting or underfitting. Meanwhile, the setting of the learning rate is crucial for the speed and stability of model convergence. Therefore, when optimizing model performance, it is essential to carefully consider the values of these two parameters and adjust them flexibly based on actual conditions to achieve the best training results.

Using the method of controlled variables, we can start with a fixed, relatively small number of epochs in the initial stage and adjust the learning rate multiple times to find an optimal learning rate. After that, we can increase the number of training epochs to further improve the results. Below, we introduce the parameter tuning method for time series classification in detail:

It is recommended to follow the method of controlled variables when debugging parameters:

1. First, fix the number of training epochs to 5 and the batch size to 16.
2. Launch three experiments based on the TimesNet_cls model with learning rates of 0.00001, 0.0001, and 0.001, respectively.
3. It can be found that Experiment 3 has the highest accuracy. Therefore, fix the learning rate at 0.001 and try increasing the number of training epochs to 30. Note: Due to the built-in early stopping mechanism for time series tasks, training will automatically stop if the validation set accuracy does not improve after 10 patience epochs. If you need to change the patience epochs for early stopping, you can manually modify the value of the `patience` hyperparameter in the configuration file.

Learning Rate Exploration Results:

| Experiment | Epochs | Learning Rate | Batch Size | Training Environment | Validation Accuracy |
|------------|--------|---------------|------------|--------------------|---------------------|
| Experiment 1 | 5 | 0.00001 | 16 | 1 GPU | 72.20% |
| Experiment 2 | 5 | 0.0001 | 16 | 1 GPU | 72.20% |
| Experiment 3 | 5 | 0.001 | 16 | 1 GPU | 73.20% |

Results of Increasing Training Epochs:

| Experiment | Epochs | Learning Rate | Batch Size | Training Environment | Validation Accuracy |
|------------|--------|---------------|------------|--------------------|---------------------|
| Experiment 3 | 5 | 0.001 | 16 | 1 GPU | 73.20% |
| Experiment 4 | 30 | 0.001 | 16 | 1 GPU | 75.10% |

## 6. Production Line Testing
Set the model directory to the trained model for testing, using the [test file](https://paddle-model-ecology.bj.bcebos.com/paddlex/PaddleX3.0/doc_images/practical_tutorial/timeseries_classification/test.csv) to perform predictions:

```bash
python main.py -c paddlex/configs/ts_classification/TimesNet_cls.yaml \
    -o Global.mode=predict \
    -o Predict.model_dir="./output/inference" \
    -o Predict.input="https://paddle-model-ecology.bj.bcebos.com/paddlex/PaddleX3.0/doc_images/practical_tutorial/timeseries_classification/test.csv"
```
Similar to model training and evaluation, the following steps are required:

* Specify the path to the `.yaml` configuration file of the model (here it is `TimesNet_cls.yaml`)
* Specify the mode as model inference prediction: `-o Global.mode=predict`
* Specify the path to the model weights: `-o Predict.model_dir="./output/inference"`
* Specify the path to the input data: `-o Predict.input="..."`

Other related parameters can be set by modifying the fields under `Global` and `Predict` in the `.yaml` configuration file. For details, refer to [PaddleX Time Series Task Model Configuration File Parameter Description](../module_usage/instructions/config_parameters_time_series_en.md).

## 7. Development Integration/Deployment
If the general time series classification pipeline meets your requirements for inference speed and accuracy, you can directly proceed with development integration/deployment.

1. If you need to directly apply the general time series classification pipeline in your Python project, you can refer to the following sample code:

```
from paddlex import create_pipeline
pipeline = create_pipeline(pipeline="ts_classification")
output = pipeline.predict("pre_ts.csv")
for res in output:
    res.print() # 打印预测的结构化输出
    res.save_to_csv("./output/") # 保存csv格式结果
```

For more parameters, please refer to the [Time Series Classification Pipeline Usage Tutorial](../pipeline_usage/tutorials/time_series_pipelines/time_series_classification_en.md)

2. Additionally, PaddleX's time series classification pipeline also offers a service-oriented deployment method, detailed as follows:

Service-Oriented Deployment: This is a common deployment form in actual production environments. By encapsulating the inference functionality as services, clients can access these services through network requests to obtain inference results. PaddleX supports users in achieving service-oriented deployment of pipelines at low cost. For detailed instructions on service-oriented deployment, please refer to the [PaddleX Service-Oriented Deployment Guide](../pipeline_deploy/service_deploy_en.md).
You can choose the appropriate method to deploy your model pipeline based on your needs, and proceed with subsequent AI application integration.
