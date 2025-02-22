# Running IBM LLMs in VS Code

Developers can now plug in AI large language models (LLM) directly into their VS code and other IDEs. There are at least four possible options in terms of how LLMs are used in the IDEs.

1. Running LLMs locally. This option is self sufficient, but it requires GPUs on your computer.
2. Running LLMs in a business environment. This option works well for business developers and requires authentication.
3. Running LLMs in accounts hosted in public cloud providers, e.g. IBM, Microsoft, Amazon, Google, etc.
4. Running LLMs in a proxy service, e.g. [OpenRouter](https://openrouter.ai)

This document covers the first two options.

## Download Continue extension

[Continue](https://www.continue.dev/) is an open-source AI code assistant that supports both VS code and JetBrains. 

## Connecting LLMs running locally for VS Code

My IBM colleague, Gabe Goodhart, has put together a detailed tutoria on how to [Build a local AI co-pilot using IBM Granite Code, Ollama, and Continue](https://developer.ibm.com/tutorials/awb-local-ai-copilot-ibm-granite-code-ollama-continue/)

Check [Continue's supported LLM providers](https://docs.continue.dev/setup/select-provider).

Also, check out the 3-minute video by Nevyan Neykov, [VSCode: install AI code helper IBM-granite](https://www.youtube.com/watch?v=VJvjgIx0a0I)


## Connecting LLMs running in IBM Cloud or On-prem for VS Code

Continue supports IBM watsonx LLMs natively whether they are available in the cloud (saas) or software (on-prem deployment). To add a model, from the Continue panel, select WatsonX and provide required parameters, e.g. url, api key and project id. The configuration is saved in config.json automatically.

![IBM watsonx LLMs](media/ibm-llms.png)

For the latest on IBM Cloud Pak for Data and watsonx.ai authentication, check [IBM watsonx.ai APIClient](https://ibm.github.io/watsonx-ai-python-sdk/base.html#credentials.Credentials)


Alternatively, you can install [LiteLLM](https://docs.litellm.ai/docs/#basic-usage and run the service locally. Or you can deploy a [docker container](https://docs.litellm.ai/docs/proxy/deploy).

```
pip install 'litellm'
#pip install 'litellm[proxy]'
litellm --model watsonx/ibm/granite-20b-code-instruct
```
By default, the service is running at http:0.0.0.0:4000. 

In the sample Continue config.json below, fill the curly brackets with the details as shown for the model, "watsonx/ibm/granite-20b-code-instruct". 

```
{
  "models": [
    {
      "title": "Local Granite-Code20B",
      "provider": "ollama",
      "model": "granite-code:20b"
    },
    {
      "title": "IBM LLM in Cloud",
      "provider": "openai",
      "model": "watsonx/ibm/granite-20b-code-instruct",
      "apiKey": "EMPTY",
      "apiBase": "http://0.0.0.0:4000"
    }
  ],
  "customCommands": [
    {
      "name": "test",
      "prompt": "{{{ input }}}\n\nWrite a comprehensive set of unit tests for the selected code. It should setup, run tests that check for correctness including important edge cases, and teardown. Ensure that the tests are complete and sophisticated. Give the tests just as chat output, don't edit any file.",
      "description": "Write unit tests for highlighted code"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Starcoder2 3b",
    "provider": "ollama",
    "model": "starcoder2:3b"
  },
  "allowAnonymousTelemetry": false
}
```

The next step is to define a few [environment variables]((https://docs.litellm.ai/docs/providers/watsonx)) for watsonx.ai authentication.

```
export WATSONX_URL=https://us-south.ml.cloud.ibm.com
export WATSONX_APIKEY=xxx
export WX_PROJECT_ID=xxx
# For custom models in your deployment, specify the deployment space id
export WATSONX_DEPLOYMENT_SPACE_ID=xxx
```

The new LLM, `granite-20b-code-instruct` is now available to use in VS Code. You can check the APIs in the browser, `http://0.0.0.0:4000`

## Connecting to self-hosted LLMs for VS Code

Check the supported [providers](https://docs.continue.dev/setup/select-provider) before going with the custom provider option.

More detail on [self hosting models](https://docs.continue.dev/setup/configuration#self-hosting-an-open-source-model). 

```
{
  "models": [
    {
      "title": "IBM LLM - ",
      "provider": "IBM",
      "model": "ibm/granite-20b-code-instruct",
      "requestOptions": {
        "headers": {
          "username": "xxx",
          "password": "xxx",
          "project_id": "xxx",
          "instance_id": "openshift",
          "version": "5.0"
        }
      }
    }
  ]
}
```

The next step is to define a [Custom LLM Provider](https://docs.continue.dev/setup/configuration#defining-a-custom-llm-provider).

```
export function modifyConfig(config: Config){
    return config;
}
```

You can find an implementation for [watsonx.ai in the pull request](https://github.com/NoeSamaille/continue-watsonx/blob/main/src/watsonx.ts).


## Use LLMs in VS Code

To use a LLM in VS Code, press "CTRL L", or "Command L" on Mac. You should see the selected LLM in use. If you have multiple LLMs configured, toggle between LLMs, whether they are running locally or remotely.

For example, type "Show me an example of connecting elasticserach in python" in the Continue pane, and you will see sample python code. Ask a follow up question, "show me how to use elasticsearch authentication in python". You should see something like this when IBM "granite-code:20b" is used.

![Example of using LLM in Continue](media/use-llm-continue.png)

## Acknowledgement

Many thanks to my colleagues, Travis Jeanneret and Will Hawkins, for sharing info regarding running LLMs in VS Code.