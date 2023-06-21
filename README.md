## GPT4ALL-Python-API



### Description

GPT4ALL-Python-API is an API for the GPT4ALL project. It provides an interface to interact with GPT4ALL models using Python.

### Features
- Possibility to list and download new models, saving them in the default directory of gpt4all GUI.
- Possibility to set a default model when initializing the class.
  - The model is loaded once and then reused.
  - The model can be set through the environment variable `DEFAULT_MODEL` in the dotenv file.
  - The model can be set through the `model` field in the request body or in the OpenAI library.
  - reload model instance over reload field in request
- Implemented the prompt_template, prompt_batch_size, and repeat_last_n via API.
- Methods included in the default API of gpt4all GUI.

### Requirements

- Python 3.8
- uvicorn
- gpt4all
- python-dotenv
- fastapi

### Installation

To install the GPT4ALL-Python-API, follow these steps:
Tip: use virtualenv, miniconda or your favorite virtual environment to install packages and run the project.

1. Install Python using Anaconda or Miniconda.
2. Create a virtual environment with Python 3.8.
3. Install the required libraries using the following command:
   ```
   pip install uvicorn gpt4all python-dotenv fastapi
   ```


### Setting up GPT4ALL Backend on Windows

If you are using Windows, please follow these instructions to set up the GPT4ALL backend using the MinGW64 compiler:

1. Clone the GPT4ALL repository:
   ```
   git clone --recurse-submodules https://github.com/nomic-ai/gpt4all
   ```
   
2. Configure llmodel:
   ```
   cd gpt4all/gpt4all-backend/
   mkdir build
   cd build
   cmake ..
   cmake --build . --parallel
   ```
   Make sure that `libllmodel.*` files exist in `gpt4all-backend/build`.

3. Configure the Python package:
   ```
   cd ../../gpt4all-bindings/python
   pip install -e .
   ```

### Usage

To run the project, follow these steps:

1. Run the following command:
   ```bash
   uvicorn inference:app --reload
   ```
   This will start the server using the `inference.py` file as the entry point and `app` as the FastAPI application variable.


1. Open your web browser and navigate to `http://localhost:8000/v1/models` to access the API available models.


## API Endpoints 

This documentation provides an overview of the available endpoints in the FastAPI application.

### POST /v1/completions

This endpoint is used to generate text completions based on the provided JSON payload.
#### Request Body

The request to the `/v1/completions` endpoint should include the following parameters in the JSON payload:

- `model` (optional): The name of the GPT4ALL model to use. If not provided, the default model will be used.
- `prompt` (optional): The prompt text as a string. If not provided or empty, an error response will be returned.
- `max_tokens` (optional): The maximum number of tokens to generate in the completion. Default value is 16.
- `temperature` (optional): The temperature value for the text generation. Default value is 1.0.
- `top_p` (optional): The top-p (nucleus) sampling probability. Default value is 1.0.
- `prompt_batch_size` (optional): The batch size for processing prompts. Default value is 128.
- `top_k` (optional): The top-k sampling value. Default value is 40.
- `n` (optional): The number of completions to generate. Not implemented.
- `thread_count` (optional): The number of threads to use for processing. Not implemented.
- `repeat_penality` (optional): The repetition penalty for the generated text. Default value is 1.18.
- `repeat_last_n` (optional): The number of tokens to consider for the repetition penalty. Default value is 64.
- `echo` (optional): Flag indicating whether to echo the prompt in the response. Default value is `False`.
- `reload` (optional): Flag indicating that the model must be loaded again thus eliminating the possible history. Default value is `False`.
- `prompt_template` (optional): The template to use for the prompt. If not provided, a default template will be used.

Example Request Body for text rewrite:
```json
{
    "prompt": "Only strings is allowed",
    "max_tokens": 50,
    "temperature": 0.28,
    "top_p": 0.1,
    "top_k": 40,
    "prompt_batch_size": 128,
    "repeat_penality": 1.18,
    "repeat_last_n": 64,
    "prompt_template": "\n### Instruction:\nParaphrase and Expand the text below.\n### Text:\n%1\n### Response:\n"
}
```

Please note that some parameters are not yet implemented and may not have any effect on the generated completions.

#### Response

If the request is successful and the prompt is not empty, the API will return a JSON response with the generated text and other information.

Example Response:
```json
{
    "id": "91c88c7886f24449ad8399b7782f56db",
    "object": "text_completion",
    "created": 1687357029.844655,
    "model": "ggml-nous-gpt4-vicuna-13b",
    "choices": [
        {
            "text": "lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris",
            "index": 0,
            "logprobs": null,
            "finish_reason": "stop",
            "references": []
        }
    ],
    "usage": {
        "prompt_tokens": 50,
        "completion_tokens": 7,
        "total_tokens": 27,
        "thread_count": 4,
        "prompt_template": "\n### Instruction:\nParaphrase and Expand the text below.\n### Text:\n%1\n### Response:\n"
    }
}
```

If the prompt is empty or not provided, an error response will be returned with the following structure:

```json
{
  "error": {
    "message": "empty prompt",
    "type": "invalid_request_error",
    "param": null,
    "code": null
  }
}
```

Please note that some parameters mentioned in the code (`n`, `thread_count`, and `echo`) are marked as "not implemented" and do not have any effect on the current implementation of the endpoint.

### POST /v1/chat/completions

Not implemented yet.
 
### GET /v1/models

This endpoint returns a list of available models.

**Request**

- Method: GET
- Path: /v1/models

**Response**

- Status Code: 200 (OK)
- Content Type: application/json
- Example Response:
  ```json
  {
    "object": "list",
    "data": [
      "model1",
      "model2",
      "model3"
    ]
  }
  ```

### GET /v1/models/{model_name}

This endpoint retrieves or downloads a specific model.

**Request**

- Method: GET
- Path: /v1/models/{model_name}

**Response**

- Status Code: 200 (OK)
- Content Type: application/json
- Example Response:
  ```json
  {
    "object": "list",
    "data": "Model downloaded successfully."
  }
  ```


### OpenAPI Wrapper Example

Example of usage with the OpenAPI library to create queries.


```python

import openai

# your uvicorn server addr, default is http://localhost:8000
openai.api_base = "http://localhost:8000/v1"
#openai.api_base = "https://api.openai.com/v1"

openai.api_key = "not needed for a local LLM"

# Set up the prompt and other parameters for the API request
prompt = "Who is Michael Jordan?"

# model = "gpt-3.5-turbo"
#model = "mpt-7b-chat"
model = "gpt4all-j-v1.3-groovy"

# Make the API request
response = openai.Completion.create(
    model=model,
    prompt=prompt,
    max_tokens=50,
    temperature=0.28,
    top_p=0.95,
    n=1,
    echo=True,
    stream=False
)

# Print the generated completion
print(response)

```
### Development Environment

We recommend using PyCharm to set up and manage your development environment. It provides an integrated development environment with powerful tools for Python development.

Make sure to activate your virtual environment and configure PyCharm to use the correct Python interpreter.

For more information, refer to the [PyCharm documentation](https://www.jetbrains.com/pycharm/).

### Additional Resources

- [GPT4ALL GitHub Repository](https://github.com/nomic-ai/gpt4all) 
- [GPT4ALL Python API Library](https://github.com/nomic-ai/gpt4all/tree/main/gpt4all-bindings/python)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [UVicorn Documentation](https://www.uvicorn.org/)

<a href="https://github.com/nomic-ai/"><img src="https://gpt4all.io/nomic_logo.png"  width="60"  ></a>

Please note that this project is still under development, and additional documentation and features will be added in the future. Feel free to contribute to the project and provide feedback.