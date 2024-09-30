[简体中文](OCR.md) | English

# General OCR Pipeline Usage Tutorial

## 1. Introduction to OCR Pipeline
OCR (Optical Character Recognition) is a technology that converts text in images into editable text. It is widely used in document digitization, information extraction, and data processing. OCR can recognize printed text, handwritten text, and even certain types of fonts and symbols.

The General OCR Pipeline is designed to solve text recognition tasks, extracting text information from images and outputting it in text form. PP-OCRv4 is an end-to-end OCR system that achieves millisecond-level text content prediction on CPUs, reaching state-of-the-art (SOTA) performance in open-source projects for general scenarios. Based on this project, developers from academia, industry, and research have rapidly deployed various OCR applications across fields such as general use, manufacturing, finance, transportation, and more.

![](/tmp/images/pipelines/ocr/01.png)

**The General OCR Pipeline comprises a text detection module and a text recognition module**, each containing several models. The specific models to use can be selected based on the benchmark data provided below. **If you prioritize model accuracy, choose models with higher accuracy. If you prioritize inference speed, choose models with faster inference. If you prioritize model size, choose models with smaller storage requirements.**

<details>
   <summary> 👉Model List Details</summary>

<table>
  <tr>
    <th>Pipeline Module</th>
    <th>Specific Model</th>
    <th>Accuracy</th>
    <th>GPU Inference Time (ms)</th>
    <th>CPU Inference Time</th>
    <th>Model Size (M)</th>
  </tr>
  <tr>
    <td rowspan="2">Text Detection</td>
    <td>PP-OCRv4_mobile_det</td>
    <td>77.79%</td>
    <td>2.719474</td>
    <td>79.1097</td>
    <td>15</td>
  </tr>
  <tr>
    <td>PP-OCRv4_server_det</td>
    <td>82.69%</td>
    <td>22.20346</td>
    <td>2662.158</td>
    <td>198</td>
  </tr>
  <tr>
    <td rowspan="5">Text Recognition</td>
    <td>PP-OCRv4_mobile_rec</td>
    <td>78.20%</td>
    <td>2.719474</td>
    <td>79.1097</td>
    <td>15</td>
  </tr>
  <tr>
    <td>PP-OCRv4_server_rec</td>
    <td>79.20%</td>
    <td>22.20346</td>
    <td>2662.158</td>
    <td>198</td>
  </tr>
  <tr>
    <td>ch_RepSVTR_rec</td>
    <td>65.07%</td>
    <td>-</td>
    <td>-</td>
    <td>22.1 M</td>
  </tr>
  <tr>
    <td>ch_SVTRv2_rec</td>
    <td>68.81%</td>
    <td> - </td>
    <td> - </td>
    <td>73.9 M</td>
  </tr>
</table>

**Note: The accuracy metric for text detection models is Hmean(%), and for text recognition models, it is Accuracy(%). All GPU inference times are based on an NVIDIA Tesla T4 machine with FP32 precision. CPU inference speeds are based on an Intel(R) Xeon(R) Gold 5117 CPU @ 2.00GHz with 8 threads and FP32 precision.**

</details>

## 2. Quick Start
PaddleX provides pre-trained models for the OCR Pipeline, allowing you to quickly experience its effects. You can try the General OCR Pipeline online or locally using command line or Python.

### 2.1 Online Experience
You can [experience the General OCR Pipeline online](https://aistudio.baidu.com/community/app/91660/webUI?source=appMineRecent) using the official demo images for recognition, for example:

![](/tmp/images/pipelines/ocr/02.png)

If you are satisfied with the pipeline's performance, you can directly integrate and deploy it. You can download the deployment package from the cloud or use the [local experience method in Section 2.2](#3-Development-and-Deployment). If not satisfied, you can also use your private data to **fine-tune the models in the pipeline online**.

### 2.2 Local Experience
> ❗ Before using the General OCR Pipeline locally, ensure you have installed the PaddleX wheel package following the [PaddleX Installation Guide](../../../installation/installation_en.md).

#### 2.2.1 Command Line Experience
* Experience the OCR Pipeline with a single command:

Experience the image anomaly detection pipeline with a single command，Use the [test file](https://paddle-model-ecology.bj.bcebos.com/paddlex/imgs/demo_image/general_ocr_002.png), and replace `--input` with the local path to perform prediction.

```bash
paddlex --pipeline OCR --input general_ocr_002.png --device gpu:0
```
Parameter explanations:

```
--pipeline: The name of the pipeline, here it is OCR.
--input: The local path or URL of the input image to be processed.
--device: The GPU index to use (e.g., gpu:0 for the first GPU, gpu:1,2 for the second and third GPUs). You can also choose to use CPU (--device cpu).
```

When executing the above command, the default OCR Pipeline configuration file is loaded. If you need to customize the configuration file, you can use the following command to obtain it:

<details>
   <summary> 👉 Click to expand</summary>

```bash
paddlex --get_pipeline_config OCR
```

After execution, the OCR Pipeline configuration file will be saved in the current directory. If you wish to customize the save location, you can execute the following command (assuming the custom save location is `./my_path`):

```bash
paddlex --get_pipeline_config OCR --config_save_path ./my_path
```

After obtaining the Pipeline configuration file, replace `--pipeline` with the configuration file's save path to make the configuration file effective. For example, if the configuration file is saved as `./ocr.yaml`, simply execute:

```bash
paddlex --pipeline ./ocr.yaml --input general_ocr_002.png
```

Here, parameters such as `--model` and `--device` do not need to be specified, as they will use the parameters in the configuration file. If parameters are still specified, the specified parameters will take precedence.

</details>

#### 2.2.2 Integration via Python Script
* Quickly perform inference on the production line with just a few lines of code, taking the general OCR production line as an example:

```python
from paddlex import create_pipeline

pipeline = create_pipeline(pipeline="ocr")

output = pipeline.predict("pre_image.jpg")
for batch in output:
    for item in batch:
        res = item['result']
        res.print()
        res.save_to_img("./output/")
        res.save_to_json("./output/")
```
> ❗ The results obtained from running the Python script are the same as those from the command line.

The Python script above executes the following steps:

（1）Instantiate the OCR production line object using `create_pipeline`: Specific parameter descriptions are as follows:

| Parameter | Description | Type | Default |
|-|-|-|-|
|`pipeline`| The name of the production line or the path to the production line configuration file. If it is the name of the production line, it must be supported by PaddleX. |`str`|None|
|`device`| The device for production line model inference. Supports: "gpu", "cpu". |`str`|`gpu`|
|`enable_hpi`| Whether to enable high-performance inference, only available if the production line supports it. |`bool`|`False`|

（2）Invoke the `predict` method of the OCR production line object for inference prediction: The `predict` method parameter is `x`, which is used to input data to be predicted, supporting multiple input methods, as shown in the following examples:

| Parameter Type | Parameter Description |
|---------------|-----------------------------------------------------------------------------------------------------------|
| Python Var    | Supports directly passing in Python variables, such as numpy.ndarray representing image data. |
| str         | Supports passing in the path of the file to be predicted, such as the local path of an image file: `/root/data/img.jpg`. |
| str           | Supports passing in the URL of the file to be predicted, such as the network URL of an image file: [Example](https://paddle-model-ecology.bj.bcebos.com/paddlex/imgs/demo_image/general_ocr_002.png). |
| str           | Supports passing in a local directory, which should contain files to be predicted, such as the local path: `/root/data/`. |
| dict          | Supports passing in a dictionary type, where the key needs to correspond to a specific task, such as "img" for image classification tasks. The value of the dictionary supports the above types of data, for example: `{"img": "/root/data1"}`. |
| list          | Supports passing in a list, where the list elements need to be of the above types of data, such as `[numpy.ndarray, numpy.ndarray], ["/root/data/img1.jpg", "/root/data/img2.jpg"], ["/root/data1", "/root/data2"], [{"img": "/root/data1"}, {"img": "/root/data2/img.jpg"}]`. |

（3）Obtain the prediction results by calling the `predict` method: The `predict` method is a `generator`, so prediction results need to be obtained through iteration. The `predict` method predicts data in batches, so the prediction results are in the form of a list.

（4）Process the prediction results: The prediction result for each sample is of `dict` type and supports printing or saving to files, with the supported file types depending on the specific pipeline. For example:

| Method         | Description                     | Method Parameters |
|--------------|-----------------------------|--------------------------------------------------------------------------------------------------------|
| print        | Prints results to the terminal  | `- format_json`: bool, whether to format the output content with json indentation, default is True;<br>`- indent`: int, json formatting setting, only valid when format_json is True, default is 4;<br>`- ensure_ascii`: bool, json formatting setting, only valid when format_json is True, default is False; |
| save_to_json | Saves results as a json file   | `- save_path`: str, the path to save the file, when it's a directory, the saved file name is consistent with the input file type;<br>`- indent`: int, json formatting setting, default is 4;<br>`- ensure_ascii`: bool, json formatting setting, default is False; |
| save_to_img  | Saves results as an image file | `- save_path`: str, the path to save the file, when it's a directory, the saved file name is consistent with the input file type; |

If you have a configuration file, you can customize the configurations of the image anomaly detection pipeline by simply modifying the `pipeline` parameter in the `create_pipeline` method to the path of the pipeline configuration file.

For example, if your configuration file is saved at `./my_path/ocr.yaml`, you only need to execute:

```python
from paddlex import create_pipeline
pipeline = create_pipeline(pipeline="./my_path/ocr.yaml")
output = pipeline.predict("general_ocr_002.png")
for res in output:
    res.print(json_format=False)
    res.save_to_img("./output/")
    res.save_to_json("./output/res.json")
```

## 3. Development Integration/Deployment
If the general OCR pipeline meets your requirements for inference speed and accuracy, you can proceed directly with development integration/deployment.

If you need to apply the general OCR pipeline directly in your Python project, refer to the example code in [2.2.2 Python Script Integration](#222-python-script-integration).

Additionally, PaddleX provides three other deployment methods, detailed as follows:

🚀 **High-Performance Deployment**: In actual production environments, many applications have stringent standards for the performance metrics of deployment strategies (especially response speed) to ensure efficient system operation and smooth user experience. To this end, PaddleX provides high-performance inference plugins aimed at deeply optimizing model inference and pre/post-processing for significant end-to-end speedups. For detailed high-performance deployment procedures, refer to the [PaddleX High-Performance Deployment Guide](../../../pipeline_deploy/high_performance_deploy_en.md).

☁️ **Service-Oriented Deployment**: Service-oriented deployment is a common deployment form in actual production environments. By encapsulating inference functions as services, clients can access these services through network requests to obtain inference results. PaddleX supports users in achieving low-cost service-oriented deployment of pipelines. For detailed service-oriented deployment procedures, refer to the [PaddleX Service-Oriented Deployment Guide](../../../pipeline_deploy/service_deploy_en.md).

Below are the API references and multi-language service invocation examples:

<details>
<summary>API Reference</summary>

For all operations provided by the service:

- Both the response body and the request body for POST requests are JSON data (JSON objects).
- When the request is processed successfully, the response status code is `200`, and the response body attributes are as follows:

    | Name | Type | Description |
    |------|------|-------------|
    |`errorCode`|`integer`|Error code. Fixed as `0`.|
    |`errorMsg`|`string`|Error description. Fixed as `"Success"`.|

    The response body may also have a `result` attribute of type `object`, which stores the operation result information.

- When the request is not processed successfully, the response body attributes are as follows:

    | Name | Type | Description |
    |------|------|-------------|
    |`errorCode`|`integer`|Error code. Same as the response status code.|
    |`errorMsg`|`string`|Error description.|

Operations provided by the service:

- **`infer`**

    Obtain OCR results from an image.

    `POST /ocr`

    - Request body attributes:

        | Name | Type | Description | Required |
        |------|------|-------------|----------|
        |`image`|`string`|The URL of an image file accessible by the service or the Base64 encoded result of the image file content.|Yes|
        |`inferenceParams`|`object`|Inference parameters.|No|

        Properties of `inferenceParams`:

        | Name | Type | Description | Required |
        |------|------|-------------|----------|
        |`maxLongSide`|`integer`|During inference, if the length of the longer side of the input image for the text detection model is greater than `maxLongSide`, the image will be scaled so that the length of the longer side equals `maxLongSide`.|No|

    - When the request is processed successfully, the `result` in the response body has the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        |`texts`|`array`|Positions, contents, and scores of texts.|
        |`image`|`string`|OCR result image with detected text positions annotated. The image is in JPEG format and encoded in Base64.|

        Each element in `texts` is an `object` with the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        |`poly`|`array`|Text position. Elements in the array are the vertex coordinates of the polygon enclosing the text.|
        |`text`|`string`|Text content.|
        |`score`|`number`|Text recognition score.|

        Example of `result`:

        ```json
        {
          "texts": [
            {
              "poly": [
                [
                  444,
                  244
                ],
                [
                  705,
                  244
                ],
                [
                  705,
                  311
                ],
                [
                  444,
                  311
                ]
              ],
              "text": "Beijing South Railway Station",
              "score": 0.9
            },
            {
              "poly": [
                [
                  992,
                  248
                ],
                [
                  1263,
                  251
                ],
                [
                  1263,
                  318
                ],
                [
                  992,
                  315
                ]
              ],
              "text": "Tianjin Railway Station",
              "score": 0.5
            }
          ],
          "image": "xxxxxx"
        }
        ```

</details>

<details>
<summary>Multilingual Service Invocation Examples</summary>

<details>
<summary>Python</summary>

```python
import base64
import requests

API_URL = "http://localhost:8080/ocr"
image_path = "./demo.jpg"
output_image_path = "./out.jpg"

with open(image_path, "rb") as file:
    image_bytes = file.read()
    image_data = base64.b64encode(image_bytes).decode("ascii")

payload = {"image": image_data}

response = requests.post(API_URL, json=payload)

assert response.status_code == 200
result = response.json()["result"]
with open(output_image_path, "wb") as file:
    file.write(base64.b64decode(result["image"]))
print(f"Output image saved at {output_image_path}")
print("\nTexts:")
print(result["texts"])
```

</details>

<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include "cpp-httplib/httplib.h" // https://github.com/Huiyicc/cpp-httplib
#include "nlohmann/json.hpp" // https://github.com/nlohmann/json
#include "base64.hpp" // https://github.com/tobiaslocker/base64

int main() {
    httplib::Client client("localhost:8080");
    const std::string imagePath = "./demo.jpg";
    const std::string outputImagePath = "./out.jpg";

    httplib::Headers headers = {
        {"Content-Type", "application/json"}
    };

    std::ifstream file(imagePath, std::ios::binary | std::ios::ate);
    std::streamsize size = file.tellg();
    file.seekg(0, std::ios::beg);

    std::vector<char> buffer(size);
    if (!file.read(buffer.data(), size)) {
        std::cerr << "Error reading file." << std::endl;
        return 1;
    }
    std::string bufferStr(reinterpret_cast<const char*>(buffer.data()), buffer.size());
    std::string encodedImage = base64::to_base64(bufferStr);

    nlohmann::json jsonObj;
    jsonObj["image"] = encodedImage;
    std::string body = jsonObj.dump();

    auto response = client.Post("/ocr", headers, body, "application/json");
    if (response && response->status == 200) {
        nlohmann::json jsonResponse = nlohmann::json::parse(response->body);
        auto result = jsonResponse["result"];

        encodedImage = result["image"];
        std::string decodedString = base64::from_base64(encodedImage);
        std::vector<unsigned char> decodedImage(decodedString.begin(), decodedString.end());
        std::ofstream outputImage(outPutImagePath, std::ios::binary | std::ios::out);
        if (outputImage.is_open()) {
            outputImage.write(reinterpret_cast<char*>(decodedImage.data()), decodedImage.size());
            outputImage.close();
            std::cout << "Output image saved at " << outPutImagePath << std::endl;
        } else {
            std::cerr << "Unable to open file for writing: " << outPutImagePath << std::endl;
        }

        auto texts = result["texts"];
        std::cout << "\nTexts:" << std::endl;
        for (const auto& category : texts) {
            std::cout << category << std::endl;
        }
    } else {
        std::cout << "Failed to send HTTP request." << std::endl;
        return 1;
    }

    return 0;
}
```

</details>

<details>
<summary>Java</summary>

```java
import okhttp3.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.node.ObjectNode;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Base64;

public class Main {
    public static void main(String[] args) throws IOException {
        String API_URL = "http://localhost:8080/ocr";
        String imagePath = "./demo.jpg";
        String outputImagePath = "./out.jpg";

        File file = new File(imagePath);
        byte[] fileContent = java.nio.file.Files.readAllBytes(file.toPath());
        String imageData = Base64.getEncoder().encodeToString(fileContent);

        ObjectMapper objectMapper = new ObjectMapper();
        ObjectNode params = objectMapper.createObjectNode();
        params.put("image", imageData);

        OkHttpClient client = new OkHttpClient();
        MediaType JSON = MediaType.Companion.get("application/json; charset=utf-8");
        RequestBody body = RequestBody.Companion.create(params.toString(), JSON);
        Request request = new Request.Builder()
                .url(API_URL)
                .post(body)
                .build();

        try (Response response = client.newCall(request).execute()) {
            if (response.isSuccessful()) {
                String responseBody = response.body().string();
                JsonNode resultNode = objectMapper.readTree(responseBody);
                JsonNode result = resultNode.get("result");
                String base64Image = result.get("image").asText();
                JsonNode texts = result.get("texts");

                byte[] imageBytes = Base64.getDecoder().decode(base64Image);
                try (FileOutputStream fos = new FileOutputStream(outputImagePath)) {
                    fos.write(imageBytes);
                }
                System.out.println("Output image saved at " + outputImagePath);
                System.out.println("\nTexts: " + texts.toString());
            } else {
                System.err.println("Request failed with code: " + response.code());
            }
        }
    }
}
```

</details>

<details>
<summary>Go</summary>

```go
package main

import (
    "bytes"
    "encoding/base64"
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
)

func main() {
    API_URL := "http://localhost:8080/ocr"
    imagePath := "./demo.jpg"
    outputImagePath := "./out.jpg"

    imageBytes, err := ioutil.ReadFile(imagePath)
    if err != nil {
        fmt.Println("Error reading image file:", err)
        return
    }
    imageData := base64.StdEncoding.EncodeToString(imageBytes)

    payload := map[string]string{"image": imageData}
    payloadBytes, err := json.Marshal(payload)
    if err != nil {
        fmt.Println("Error marshaling payload:", err)
        return
    }

    client := &http.Client{}
    req, err := http.NewRequest("POST", API_URL, bytes.NewBuffer(payloadBytes))
    if err != nil {
        fmt.Println("Error creating request:", err)
        return
    }

    res, err := client.Do(req)
    if err != nil {
        fmt.Println("Error sending request:", err)
        return
    }
    defer res.Body.Close()

    body, err := ioutil.ReadAll(res.Body)
    if err != nil {
        fmt.Println("Error reading response body:", err)
        return
    }
    type Response struct {
        Result struct {
            Image      string   `json:"image"`
            Texts []map[string]interface{} `json:"texts"`
        } `json:"result"`
    }
    var respData Response
    err = json.Unmarshal([]byte(string(body)), &respData)
    if err != nil {
        fmt.Println("Error unmarshaling response body:", err)
        return
    }

    outputImageData, err := base64.StdEncoding.DecodeString(respData.Result.Image)
    if err != nil {
        fmt.Println("Error decoding base64 image data:", err)
        return
    }
    err = ioutil.WriteFile(outputImagePath, outputImageData, 0644)
    if err != nil {
        fmt.Println("Error writing image to file:", err)
        return
    }
    fmt.Printf("Image saved at %s.jpg\n", outputImagePath)
    fmt.Println("\nTexts:")
    for _, category := range respData.Result.Texts {
        fmt.Println(category)
    }
}
```

</details>

<details>
<summary>C#</summary>

```csharp
using System;
using System.IO;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Text;
using System.Threading.Tasks;
using Newtonsoft.Json.Linq;

class Program
{
    static readonly string API_URL = "http://localhost:8080/ocr";
    static readonly string imagePath = "./demo.jpg";
    static readonly string outputImagePath = "./out.jpg";

    static async Task Main(string[] args)
    {
        var httpClient = new HttpClient();

        byte[] imageBytes = File.ReadAllBytes(imagePath);
        string image_data = Convert.ToBase64String(imageBytes);

        var payload = new JObject{ { "image", image_data } };
        var content = new StringContent(payload.ToString(), Encoding.UTF8, "application/json");

        HttpResponseMessage response = await httpClient.PostAsync(API_URL, content);
        response.EnsureSuccessStatusCode();

        string responseBody = await response.Content.ReadAsStringAsync();
        JObject jsonResponse = JObject.Parse(responseBody);

        string base64Image = jsonResponse["result"]["image"].ToString();
        byte[] outputImageBytes = Convert.FromBase64String(base64Image);

        File.WriteAllBytes(outputImagePath, outputImageBytes);
        Console.WriteLine($"Output image saved at {outputImagePath}");
        Console.WriteLine("\nTexts:");
        Console.WriteLine(jsonResponse["result"]["texts"].ToString());
    }
}
```

</details>

<details>
<summary>Node.js</summary>

```js
const axios = require('axios');
const fs = require('fs');

const API_URL = 'http://localhost:8080/ocr'
const imagePath = './demo.jpg'
const outputImagePath = "./out.jpg";

let config = {
   method: 'POST',
   maxBodyLength: Infinity,
   url: API_URL,
   data: JSON.stringify({
    'image': encodeImageToBase64(imagePath)
  })
};

function encodeImageToBase64(filePath) {
  const bitmap = fs.readFileSync(filePath);
  return Buffer.from(bitmap).toString('base64');
}

axios.request(config)
.then((response) => {
    const result = response.data["result"];
    const imageBuffer = Buffer.from(result["image"], 'base64');
    fs.writeFile(outputImagePath, imageBuffer, (err) => {
      if (err) throw err;
      console.log(`Output image saved at ${outputImagePath}`);
    });
    console.log("\nTexts:");
    console.log(result["texts"]);
})
.catch((error) => {
  console.log(error);
});
```

</details>

<details>
<summary>PHP</summary>

```php
<?php

$API_URL = "http://localhost:8080/ocr";
$image_path = "./demo.jpg";
$output_image_path = "./out.jpg";

$image_data = base64_encode(file_get_contents($image_path));
$payload = array("image" => $image_data);

$ch = curl_init($API_URL);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($payload));
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$response = curl_exec($ch);
curl_close($ch);

$result = json_decode($response, true)["result"];
file_put_contents($output_image_path, base64_decode($result["image"]));
echo "Output image saved at " . $output_image_path . "\n";
echo "\nTexts:\n";
print_r($result["texts"]);

?>
```

</details>
</details>
<br/>

📱 **Edge Deployment**: Edge deployment is a method that places computing and data processing capabilities on user devices themselves, allowing devices to process data directly without relying on remote servers. PaddleX supports deploying models on edge devices such as Android. For detailed edge deployment procedures, refer to the [PaddleX Edge Deployment Guide](../../../pipeline_deploy/lite_deploy_en.md).
You can choose the appropriate deployment method based on your needs to proceed with subsequent AI application integration.


## 4. Customization and Fine-tuning
If the default model weights provided by the general OCR pipeline do not meet your requirements for accuracy or speed in your specific scenario, you can try to further fine-tune the existing models using **your own domain-specific or application-specific data** to improve the recognition performance of the general OCR pipeline in your scenario.

### 4.1 Model Fine-tuning
Since the general OCR pipeline consists of two modules (text detection and text recognition), unsatisfactory performance may stem from either module.

You can analyze images with poor recognition results. If you find that many texts are undetected (i.e., text miss detection), it may indicate that the text detection model needs improvement. You should refer to the [Customization](../../../module_usage/tutorials/ocr_modules/text_detection_en.md#customization) section in the [Text Detection Module Development Tutorial](../../../module_usage/tutorials/ocr_modules/text_detection_en.md) and use your private dataset to fine-tune the text detection model. If many recognition errors occur in detected texts (i.e., the recognized text content does not match the actual text content), it suggests that the text recognition model requires further refinement. You should refer to the [Customization](../../../module_usage/tutorials/ocr_modules/text_recognition_en.md#customization) section in the [Text Recognition Module Development Tutorial](../../../module_usage/tutorials/ocr_modules/text_recognition_en.md) and fine-tune the text recognition model.

### 4.2 Model Application
After fine-tuning with your private dataset, you will obtain local model weights files.

If you need to use the fine-tuned model weights, simply modify the pipeline configuration file by replacing the local paths of the fine-tuned model weights to the corresponding positions in the pipeline configuration file:

```python
......
Pipeline:
  det_model: PP-OCRv4_server_det  # Can be replaced with the local path of the fine-tuned text detection model
  det_device: "gpu"
  rec_model: PP-OCRv4_server_rec  # Can be replaced with the local path of the fine-tuned text recognition model
  rec_batch_size: 1
  rec_device: "gpu"
......
```

Then, refer to the command line method or Python script method in [2.2 Local Experience](#22-local-experience) to load the modified pipeline configuration file.

## 5. Multi-Hardware Support
PaddleX supports various mainstream hardware devices such as NVIDIA GPU, Kunlun XPU, Ascend NPU, and Cambricon MLU. **Simply modifying the `--device` parameter** allows seamless switching between different hardware.

For example, if you are using an NVIDIA GPU for OCR pipeline inference, the Python command would be:

```bash
paddlex --pipeline OCR --input general_ocr_002.png --device gpu:0
```
Now, if you want to switch the hardware to Ascend NPU, you only need to modify the `--device` in the Python command:

```bash
paddlex --pipeline OCR --input general_ocr_002.png --device npu:0
```

If you want to use the General OCR pipeline on more types of hardware, please refer to the [PaddleX Multi-Hardware Usage Guide](../../../installation/installation_other_devices_en.md).