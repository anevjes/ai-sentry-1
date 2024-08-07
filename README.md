# AI-Sentry Facade


![AI-Sentry-features image](/content/images/AI-Sentry-features.png)

*Ai-Sentry* is transparent python + DAPR based pluggable Generative AI Facade layer, designed to support the following features for large enterprises developing and operating Generative AI solutions:

- Cross charge back on token usage across different openAI consumers
- Request/Response async based logging with ability to toggle PII stripping of information. This level of logging is useful for many things such as legal compliance as well as assessing and replaying back request/responses against newer models to help you deal with model upgrades without affecting your existing users.
- Smarter load balancing by taking into account Azure openAI's response header load metrics and pooling of multi backends with same model capabilities
- Support streaming and non streaming responses (including logging of these)
- Extensibility of custom adapters to help you deal with SDK / API deprecations from client side - so you can provide backwards compatibility if needed.


AI-Sentry is not designed to replace existing API Gateway solutions such as Azure APIM - rather it is designed to sit between API Gateway and the openAI endpoints - providing ultimate control for your openAI solutions.

We try to perform heavy processing outside of the direct HTTP calls pipeline to minimise latency to the consumers and rely on DAPR side cars and Pub / Sub patterns to perform the work asynchronously.

Because AI-Sentry uses DAPR; the technology choices for log persistence, and message brokers is swappable out via DAPR's native [components](https://docs.dapr.io/concepts/components-concept/). Our example uses REDIS and Event Hubs as the message broker for PUB/SUB, and CosmosDB as the Log persistence store.

## High Level Design

![ISentryHighLevel image](/content/images/AI-Sentry-HighLevel.drawio.png)



## Backend Configuration

The following environment variables need to exist. How you feed them in is up to you - i.e. Kubernetes secrets, configmaps, etc...

| Name | Value | Component |
| -------- | -------- | -------- |
|  AI-SENTRY-ENDPOINT-CONFIG  | Example JSON value is located [here](/content/documentation/ai-sentry-config.json). This is used to map openai endpoints / deployments - so that when we are load balancing we are hitting group of same openAI models from the pool.  Make sure to include /openai in your endpoint url configuration. You can leverage the following [script](scripts/create-escaped-json.ps1) to help you generate JSON escaped string of this JSON.|Facade App |
|AI-SENTRY-LANGUAGE-KEY| your Congnitive Services General API Key| CosmosDB Worker |
|AI-SENTRY-LANGUAGE-ENDPOINT| your language text anlaytics or general service endpoint url| CosmosDB Worker |


## Consumer Configuration

Whatever you front AI-Sentry with e.g. Azure APIM, some other API gateway technology - you will need to supply some mandatory HTTP headers.

|HTTP HEADER NAME| HTTP HEADER VALUE|
| -------- | --------|
|ai-sentry-consumer| this can be any string - it is used to represent a consumer or a product that uses generative ai backend. We use this for logging purposes|
| ai-sentry-log-level | This toggles logging level for the actual consumer. Accepted values are: COMPLETE, PII_STRIPPING_ENABLED or DISABLED |
|ai-sentry-backend-pool| Provide the name of the pool from the AI-SENTRY-ENDPOINT-CONFIG configuration. E.g. Pool1|
|ai-sentry-adapters| Provide list of adapter names you want to run through prior to sending out the request to openai endpoint. Example: ```["SampleApiRequestTransformer","adapter2..."]```

## Getting started

For more information on setting up AI-Sentry in your environment please follow the following detailed sections.

- [Setting up CosmosDB dbs/table](/content/documentation/CosmosDBSetup.md)

- [Setting up AI-Sentry on AKS](/content/documentation/AKSDeployment.md)

- [CosmosDB Logging Schema](/content/documentation/ComsosDB-LoggingSchema.md)

- [Summary Logging Schema](/content/documentation/SummaryLog-schema.md)

- [Setting up workload identity - if you want to auth to openai backends via JWT instead of api keys](/content/documentation/Workload-identity-config.md)


## Looking for dotnet based solution?

Thankfully our colleauge Graeme Foster has published a dotnet version with similar feature sets. Please go and check it out: https://github.com/microsoft/aicentral

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft 
trademarks or logos is subject to and must follow 
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
