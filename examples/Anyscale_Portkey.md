# Portkey + Anyscale

[Portkey](https://docs.portkey.ai/) acts as an AI Gateway to Anyscale to ensure the reliability you want to provide for your users. 

Use the Anyscale API **through** Portkey for:

1. **Enhanced Logging**: Track API use with detailed insights.
2. **Production Reliability**: Automated fallbacks, load balancing, and caching.
3. **Continuous Improvement**: Collect and apply user feedback.
4. **Enhanced Fine-Tuning**: Combine logs & user feedback for targetted fine-tuning.

## Setup 

To provide you with those features, the requests will be securely proxied through your Portkey account.

To set one up:

1. [Sign up](https://portkey.ai/signup)
2. Grab the [API Key](https://docs.portkey.ai/portkey-docs/introduction/getting-started#first-obtain-your-portkey-api-key)

You are now ready to fully manage LLM Operations for your apps!

## Logs

To enable logs for your application, you simply will make call to Anyscale, just via Portkey.

Portkey Gateway URL: `https://api.portkey.ai/v1/chat/completions`

```py
import openai

PORTKEY_GATEWAY_URL = "https://api.portkey.ai/v1/chat/completions"

PORTKEY_HEADERS = {
	'Authorization': Bearer ANYSCALE_KEY',
	'Content-Type': 'application/json',
	'x-portkey-api-key': '<PORTKEY_API_KEY>', # Substitue Portkey API Key
	'x-portkey-mode': 'proxy anyscale' 		# Enable proxying to Anyscale 
}

client = openai.OpenAI(base_url=PORTKEY_GATEWAY_URL, default_headers=PORTKEY_HEADERS)

response = client.chat.completions.create(
    model="mistralai/Mistral-7B-Instruct-v0.1",
    messages=[{"role": "user", "content": "Say this is a test"}]
)

print(chat_complete.choices[0].message.content)

```

Try it out!

You will see all the logs from your Portkey dashboard in dimensions such as latency, cost, tokens and even dive deep into analytics.

## Observability
Let's enable following for our app:

* **Trace** requests with single id.
* Pass **Metadata** to filter requests. 

```py
import json

TRACE_ID = 'my_first_trace' # Any Searchable Label 

METADATA = {
    "_environment": "production",
    "_user": "userid123",
    "_organisation": "orgid123",
    "_prompt": "summarisationPrompt"
}

PORTKEY_HEADERS = {
	'Authorization': Bearer ANYSCALE_KEY',
	'Content-Type': 'application/json',
	'x-portkey-api-key': '<PORTKEY_API_KEY>',
	'x-portkey-mode': 'proxy anyscale',
	'x-portkey-trace-id': TRACE_ID, 		# Send the trace id
	'x-portkey-metadata': json.dumps(METADATA) 	# Send the metadata
}

client = openai.OpenAI(base_url=PORTKEY_GATEWAY_URL, default_headers=PORTKEY_HEADERS)

response = client.chat.completions.create(
    model="mistralai/Mistral-7B-Instruct-v0.1",
    messages=[{"role": "user", "content": "Say this is a test"}]
)

print(chat_complete.choices[0].message.content)
```

## Caching, Fallbacks, Load Balancing

Portkey's AI Gateway can apply features like fallbacks or caching at each request level. You can do that by saving _configs_ (from the portkey dashboard > configs tab) to get config identifier. This identifier can be passed to `x-portkey-config` header as a `string`. 

If we want to enable semantic caching + fallback from Llama2 to Mistral, your Portkey config would look like this:

```json
{
	"cache": "semantic",
	"mode": "fallback",
	"options": [
		{
			"provider": "anyscale", "api_key": "...",
			"override_params": {"model": "meta-llama/Llama-2-7b-chat-hf"}
		},
		{
			"provider": "anyscale", "api_key": "...",
			"override_params": {"model": "mistralai/Mistral-7B-Instruct-v0.1"}
		}
	]
}
```


Now, just send the Config key with `x-portkey-config` header:

```py

PORTKEY_HEADERS = {
	'x-portkey-api-key': 'PORTKEY_API_KEY',
	'Content-Type': 'application/json'
	'x-portkey-config': '<CONFIG_KEY>' # Identifier upon saving the configs
}

client = openai.OpenAI(base_url=PORTKEY_GATEWAY_URL, default_headers=PORTKEY_HEADERS)

response = client.chat.completions.create(
    messages=[{"role": "user", "content": "Say this is a test"}]
)

print(chat_complete.choices[0].message.content)
```

For more on Configs and other gateway feature like Load Balancing, [check out the docs.](https://docs.portkey.ai/portkey-docs/portkey-features/ai-gateway)

## Collect Feedback

Gather weighted feedback from users and improve your app:

```py
import requests
import json

PORTKEY_FEEDBACK_URL = "https://api.portkey.ai/v1/feedback" # Portkey Feedback Endpoint

PORTKEY_HEADERS = {
	"x-portkey-api-key": "PORTKEY_API_KEY",
	"Content-Type": "application/json",
}

DATA = {
	"trace_id": "my_second_trace_feedback", # Append feedback to Trace ID
	"value": 1,
	"weight": 0.5
}

response = requests.post(PORTKEY_FEEDBACK_URL, headers=PORTKEY_HEADERS, data=json.dumps(DATA))

print(response.text)
```

## Continuous Fine-Tuning

Once you start logging your requests and their feedback with Portkey, it becomes very easy to:

1. Curate & create data for fine-tuning, 
2. Schedule fine-tuning jobs, and 
3. Use the fine-tuned models!

Fine-tuning is currently enabled for select orgs - please request access on [Portkey Discord](https://discord.gg/sDk9JaNfK8) and we'll get back to you ASAP.

<img src="https://portkey.ai/blog/content/images/2023/11/fine-tune.gif" alt="header" width=600 />

### Wrap Up

Integrating Portkey with Anyscale helps you build resilient LLM apps from the get-go. With features like semantic caching, observability, load balancing, feedback, and fallbacks, you can ensure optimal performance and continuous improvement.

[Read full Portkey docs here.](https://docs.portkey.ai/)

Reach out to the Portkey team <a href="https://discord.gg/sDk9JaNfK8" target="_blank"><img src="https://img.shields.io/discord/1143393887742861333?logo=discord" alt="Discord"></a>
