# Enterprise Azure OpenAI

Repository detailing the deployment of an Enterprise Azure OpenAI reference architecture.
<br/>Link: [Azure Architecture Center - Monitor OpenAI Models](https://learn.microsoft.com/en-us/azure/architecture/example-scenario/ai/log-monitor-azure-openai)

An [advanced pattern](/advanced-logging/README.md) is available for customers using models with larger token sizes, want to perform advanced analytics on the prompts and responses, or have requirements to send these events to another type of data store.

## Key Solution Advantages:
*	<b>Comprehensive logging of Azure OpenAI model execution tracked to Source IP address.</b>  Log information includes what text users are submitting to the model as well as text being received back from the model.  This ensures models are being used responsibly within the corporate environment and within the approved use cases of the service.
*	<b>Advanced Usage and Throttling controls</b> allow fine-grained access controls for different user groups without allowing access to underlying service keys.
*	<b>High availability of the model APIs</b> to ensure user requests are met even if the traffic exceeds the limits of a single Azure OpenAI service.
*	<b>Secure use of the service</b> by ensuring role-based access managed via Azure Active Directory follows principle of least privilege.

<div>
  <video controls src="https://user-images.githubusercontent.com/47987698/232847412-e0f5cdd1-a587-457f-9365-e3caa21c8ec9.mp4" muted="false"></video>
</div>

## Reference Architecture
![img](/assets/EnterpriseAOAI-Architecture.png)

<table style="border-collapse:collapse">
  <tr>
    <td style="border: none;vertical-align: top; font-size:.9em;">1. Client applications can access Azure OpenAI endpoints to perform text generation (completions) and model training (fine-tuning) endpoints to leverage the power of large language models.<br/>
    <br/>
    2. Next-Gen Firewall Appliance (Optional) - Provides deep packet level inspection for network traffic to the OpenAI Models.
    </td>
    <td style="border: none;vertical-align: top; font-size:.9em;">3. API Management Gateway enables security controls, auditing, and monitoring of the Azure OpenAI models.  Security access is granted via AAD Groups with subscription based access permissions in APIM.  Auditing is enabled via Azure Monitor request logging for all interactions with the models.  Monitoring enables detailed AOAI model usage KPIs/Metrics.</td>
    <td style="border: none;vertical-align: top; font-size:.9em;">4. API Management Gateway connects to all Azure resources via Private Link to ensure all traffic is secured by private endpoints and contained to private network.<br/><br/>5. Multiple Azure OpenAI instances enable scale out of API usage to ensure high-availability and disaster recovery for the service.</td>
  </tr>

</table>

## Features

This project framework provides the following features:

* Enterprise logging of OpenAI usage metrics:
   * Token Usage
   * Model Usage
   * Prompt Input
   * User statistics
   * Prompt Response
* High Availability of OpenAI service with region failover.
* Integration with latest OpenAI libraries-
  *  [OpenAI](https://github.com/openai/openai-python/) 
  *  [LangChain](https://python.langchain.com/en/latest/)
  *  [Llama-index](https://gpt-index.readthedocs.io/en/latest/)

## Getting Started

### Prerequisites
- [Azure Subscription](https://azure.microsoft.com/en-us/get-started/)
- [Azure OpenAI Application](https://aka.ms/oai/access) 
  
### Installation
Provisioning artifacts, begin by provisioning the solution artifacts listed below:

-	[Azure OpenAI Cognitive Service]( https://azure.microsoft.com/en-us/products/cognitive-services/openai-service/)
-	[Azure API Management](https://azure.microsoft.com/services/api-management/)
-	[Azure Monitor](https://azure.microsoft.com/services/monitor/)

(Optional)
- Next-Gen Firewall Appliance
-	[Azure Application Gateway](https://azure.microsoft.com/services/application-gateway/)
-	[Azure Virtual Network](https://azure.microsoft.com/services/virtual-network/)

### Managed Services
-	[Azure Key Vault](https://azure.microsoft.com/services/key-vault/)
-	[Azure Storage](https://azure.microsoft.com/services/storage/)
-	[Azure Active Directory](https://azure.microsoft.com/services/active-directory/)

## Configuration

### Azure OpenAI
- To begin, provision a resource for Azure OpenAI in your preferred region: [Provision resource](https://portal.azure.com/?microsoft_azure_marketplace_ItemHideKey=microsoft_openai_tip#create/Microsoft.CognitiveServicesOpenAI)

- Once the resource is provisioned, create a deployment with model of choice: [Deploy Model](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/how-to/create-resource?pivots=web-portal#deploy-a-model)

- After the model has been deployed, go to the OpenAI studio to test your newly created model with the studio playground: [oai.azure.com/portal](oai.azure.com/portal)

- Note down Key1 from the Azure OpenAI instance by opening the Azure OpenAI instance, then from the Resource Management section of the left menu, select Keys and Endpoints.

### Azure Key Vault
Provision an Azure Key Vault Resource: [Deploy Key Vault](https://portal.azure.com/#create/Microsoft.KeyVault)

Once deployed, add Key1 from the Azure OpenAI instance as a secret: [Add a Secret](https://learn.microsoft.com/en-us/azure/key-vault/secrets/quick-create-portal)

### API Management Config

- API Management can be provisioned through Azure Portal :[Provision resource](https://learn.microsoft.com/en-us/azure/api-management/get-started-create-service-instance) 
- Once the API Management service has been provisioned, follow this documentation to configure access permissions for the APIM instance on the Azure Key Vaults secrets.

  - <b>Named Value Setup</b>
    - Follow this documentation to create a named value linked to Key1 in the Azure Key Vault created earlier: [Add a plain or secret value to API Management](https://learn.microsoft.com/en-us/azure/api-management/api-management-howto-properties?tabs=azure-portal#add-a-plain-or-secret-value-to-api-management)
    
  - <b>Backend Setup</b>
  - From the left menu in API Management select Backends, then create a new backend.
    - Configure the backend service to the endpoint of your deployed OpenAI service with /openai as the path: 
    - Example: <b>https://< yourservicename >.openai.azure.com<i>/openai</i></b>
      - [Retrieve endpoint](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/quickstart?pivots=programming-language-python#retrieve-key-and-endpoint)
    ![image](https://github.com/SOE-YoungS/openai-python-enterprise-logging/assets/95053834/00dca41d-cd4b-42dc-a296-a20517f83348)

  - Under Authorization for the backend, set a new header named "api-key" and set its value to the created named value, then save the config.
    ![image](https://github.com/SOE-YoungS/openai-python-enterprise-logging/assets/95053834/b8b4c267-792d-4769-a162-e0f3d77dfa40)
    
  - [API Import instructions](https://learn.microsoft.com/en-us/azure/api-management/import-and-publish#go-to-your-api-management-instance)
  - Open the APIM - API blade and Select the Import option for an existing API.  
  ![img](/assets/apim_config_0.0.png)
  - Select the Update option to update the API to the current OpenAI specifications.
    - Completions OpenAPI -  https://raw.githubusercontent.com/Azure/azure-rest-api-specs/main/specification/cognitiveservices/data-plane/AzureOpenAI/inference/stable/2023-05-15/inference.json
  ![img](/assets/apim_config_0.1.png)
  - (Optional) For Semantic Kernel compatibility "Update" the following Authoring API endpoints:
    - Authoring OpenAPI - https://raw.githubusercontent.com/Azure/azure-rest-api-specs/c183bb012de8e9e1d0d2e67a0994748df4747d2c/specification/cognitiveservices/data-plane/AzureOpenAI/authoring/stable/2022-12-01/azureopenai.json
- <b>For All API Operations</b>:
  - Build your API inbound policy as below.
  ![image](https://github.com/SOE-YoungS/openai-python-enterprise-logging/assets/95053834/6f76ba77-c148-46c1-ab71-e800f992e984)
  - Configure the Diagnostic Logs settings:
    - Set the sampling rate to 100%
    - Set the "Number of payload bytes to log" as the maximum.
  ![img](/assets/apim_config_3.png)

- Test API
  - Test the endpoint by providing the "deployment-id", "api-version" and a sample prompt:
    ![img](/assets/apim_config_4.png)

  
#### (Optional) Subscription Access Control
API Management allows API providers to protect their APIs from abuse and create value for different API product tiers. Use of  API Management layer to throttle incoming requests is a key role of Azure API Management. Either by controlling the rate of requests or the total requests/data transferred.
- Details for configuring APIM Layer : https://learn.microsoft.com/en-us/azure/api-management/api-management-sample-flexible-throttling

- Details for enabling Subscription based access to API's: [API Management Subscriptions](https://learn.microsoft.com/en-us/azure/api-management/api-management-subscriptions)
  - Note: To enable API usage via existing libraries, such as Semantic Kernel etc... you can also adjust the "Subscription" settings for the API to the following,
    <br/>![image](https://github.com/SOE-YoungS/openai-python-enterprise-logging/assets/95053834/97a30a2d-bb71-456c-ba8c-f376c5c255fa)
    <br/>In the calling client (using the library), you then set the "OpenAI / Azure OpenAI" URL & Key to the values for your API base URL / APIM subscription key.

### Logging OpenAI completions
- Once the API Management layer has been configured, you can configure existing OpenAI python code to use the API layer by adding the subscription key parameter to the completion request:
Example:
```python
import openai

openai.api_type = "azure"
openai.api_base = "https://xxxxxxxxx.azure-api.net/" # APIM Endpoint
openai.api_version = "2023-05-15"
openai.api_key = "APIM SUBSCRIPTION KEY" #DO NOT USE ACTUAL AZURE OPENAI SERVICE KEY


response = openai.Completion.create(engine="modelname",  
                                    prompt="prompt text", temperature=1,  
                                    max_tokens=200,  top_p=0.5,  
                                    frequency_penalty=0,  
                                    presence_penalty=0,  
                                    stop=None) 

```

</code>

## Demo

- Once OpenAI requests begin to log to the Azure Monitor service, you can begin to analyze the service usage using Log Analytics queries.
  - [Log Analytics Tutorial](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-tutorial)
- The table should be named <b>"ApiManagementGatewayLogs"</b>
- The <b>BackendResponseBody</b> field contains the json response from the OpenAI service which includes the text completion as well as the token and model information.
- Example query to identify token usage by ip and model:
```kusto
ApiManagementGatewayLogs
| where tolower(OperationId) in ('completions_create','chatcompletions_create')
| where ResponseCode  == '200'
| extend modelkey = substring(parse_json(BackendResponseBody)['model'], 0, indexof(parse_json(BackendResponseBody)['model'], '-', 0, -1, 2))
| extend model = tostring(parse_json(BackendResponseBody)['model'])
| extend prompttokens = parse_json(parse_json(BackendResponseBody)['usage'])['prompt_tokens']
| extend completiontokens = parse_json(parse_json(BackendResponseBody)['usage'])['completion_tokens']
| extend totaltokens = parse_json(parse_json(BackendResponseBody)['usage'])['total_tokens']
| extend ip = CallerIpAddress
| where model !=  ''
| summarize
    sum(todecimal(prompttokens)),
    sum(todecimal(completiontokens)),
    sum(todecimal(totaltokens)),
    avg(todecimal(totaltokens))
    by ip, model
```
![img](/assets/monitor_0.png)
- Example query to monitor prompt completions: 
```kusto
ApiManagementGatewayLogs
| where tolower(OperationId) in ('completions_create','chatcompletions_create')
| where ResponseCode  == '200'
| extend model = tostring(parse_json(BackendResponseBody)['model'])
| extend prompttokens = parse_json(parse_json(BackendResponseBody)['usage'])['prompt_tokens']
| extend prompttext = substring(parse_json(parse_json(BackendResponseBody)['choices'])[0], 0, 100)
```
![img](/assets/monitor_1.png)

## Resources
- Azure API Management Policies for Azure OpenAI: https://github.com/mattfeltonma/azure-openai-apim

## Frequently Asked Questions
- Where is the "Deploy to Azure" button?
  - In our experience, most enterprise cloud administrators first need to understand the solution before deploying it into an enterprise environment.  The steps in this repo show how each component is deployed and configured so that they can be integrated into your existing deployment scripts.  We do have [bicep templates](deploy) available to accelerate your development once you are familiar with the architecture.
- Does the solution work with Private Endpoints?
  - Yes, to configure the solution to work with private endpoints you will need to:
    - Configure your OpenAI instance to use a [private endpoint](https://learn.microsoft.com/en-us/azure/cognitive-services/cognitive-services-virtual-networks?tabs=portal#use-private-endpoints).
    - Ensure API Management can resolve the private endpoint, if they are in different virtual networks this may require [vnet link](https://learn.microsoft.com/en-us/azure/dns/private-dns-virtual-network-links)
    - Configure API Management to use [internal networking](https://learn.microsoft.com/en-us/azure/api-management/api-management-using-with-internal-vnet?tabs=stv2#enable-vnet-connection)
    - Ensure that API Management endpoints are accessible by your client [link](https://learn.microsoft.com/en-us/azure/api-management/private-endpoint)
- How do I secure my Azure OpenAI endpoints once this solution is deployed?
  - Option 1: Rotate all OpenAI Service keys once API Management is configured. <br>![image](https://github.com/SOE-YoungS/openai-python-enterprise-logging/assets/95053834/ea6fd8b6-3dff-461e-9bd6-ba267e2e2430)
  - Option 2: Disable key based access to Azure OpenAI Instance
    - Will impact Azure OpenAI Studio tool
