# Azure Functions

Azure Functions is a serverless solution that allows you to write less code, maintain less infrastructure, and save on costs.

Functions provides a comprehensive set of event-driven triggers and bindings that connect your functions to other services without having to write extra code.

## Use Cases

###  Process file uploads

Run code when a file is uploaded or changed in blob storage.

Trigger Azure Functions on blob containers using an event subscription:

https://learn.microsoft.com/en-us/azure/azure-functions/functions-event-grid-blob-trigger?pivots=programming-language-java

### Process data in real time

Capture and transform data from event and IoT source streams on the way to storage.

Azure Event Hubs trigger for Azure Functions:

https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-event-hubs-trigger?pivots=programming-language-java&tabs=python-v2%2Cisolated-process%2Cnodejs-v4%2Cfunctionsv2%2Cextensionv5

Azure Functions supports trigger and output bindings for Event Hubs.

```
import logging
import azure.functions as func

app = func.FunctionApp()

@app.function_name(name="EventHubTrigger1")
@app.event_hub_message_trigger(arg_name="myhub", 
                               event_hub_name="<EVENT_HUB_NAME>",
                               connection="<CONNECTION_SETTING>") 
def test_function(myhub: func.EventHubEvent):
    logging.info('Python EventHub trigger processed an event: %s',
                myhub.get_body().decode('utf-8'))
```

### Infer on data models

Pull text from a queue and present it to various AI services for analysis and classification.

### Build a scalable web API 

Implement a set of REST endpoints for your web applications using HTTP triggers.

An HTTP triggered function defines an HTTP endpoint. These endpoints run function code that can connect to other services directly or by using binding extensions. You can compose the endpoints into a web-based API.

```
import azure.functions as func
import logging

app = func.FunctionApp()

@app.function_name(name="HttpTrigger1")
@app.route(route="hello", auth_level=func.AuthLevel.ANONYMOUS)
def test_function(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')
    return func.HttpResponse(
        "This HTTP triggered function executed successfully.",
        status_code=200
        )
```

### Build a serverless workflow 

Create an event-driven workflow from a series of functions using Durable Functions.

### Create reliable message systems 

Process message queues using Queue Storage, Service Bus, or Event Hubs.