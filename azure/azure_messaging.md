# Event-driven architecture on azure

https://www.youtube.com/watch?v=FVOhLqE9fzw

## Kinds of messages

__Job__ or __Commands__: someone gives a job to do to somebody and you expect them to do it (purchase order, delivery order). Happens exactly once. Measure progress. There is a contract.

__Events__: has happened already. Facts (image uploaded) and events streams (observations) or reports in state changes. Usually take an aggregation of events (sensor measurements) over time and act on it.

Events travel through many services / software components. The middleware (standarization) should have some way to move the same event.

## Event Grid

Scalable, fully managed Pub Sub message distribution service that offers flexible message consumption patterns using the MQTT and HTTP protocols.
You can build data pipelines with device data, integrate applications, and build event-driven serverless architectures.
Event Grid can be configured to send events to subscribers (push delivery) or subscribers can connect to Event Grid to read events (pull delivery).

DISCRETE FACTS, APPLICATIONS REACT TO EVENTS (automate business process and workflow, react to status change, integrate with external service)

serverless, service-to-service communication, dynamically scalable, at least once delivery, schema, keeps trying to deliver for 24 hours, PUSH or PULL-based.

Publish-subscribe messaging model - Communicate efficiently using one-to-many, many-to-one, and one-to-one messaging patterns.

- __ingestion and push-style distribution of descrete events__
- __insulates publisher of event from failures of consumers__
- __manages delivery of events__
- reactive programming
- sms push notification
- IoT devices
- user clicks
- routing events
- azure automation
- application integration
- millions per second
- descrete events to serverless workloads

### Featues

__Push__: push delivery mechanism sends data to destinations that include your own application webhooks and Azure services. Push delivery features a 24-hour retry mechanism with exponential backoff to make sure events are delivered.

Your own __application events__ - Use Event Grid to route, filter, and reliably deliver custom events from your app.

__Partner events__– Subscribe to your partner SaaS provider events and process them on Azure.

__Advanced filtering__ – Filter on event type or other event attributes to make sure your event handlers or consumer apps receive only relevant events.

__Source of data__: 
* MQTT clients’ data
* HTTP: Azure services, Your custom applications, External partner (SaaS) systems

__High throughput__- Build high-volume integrated solutions with Event Grid.

### Concepts

#### Event

An __event__ is the smallest amount of information that fully describes something that happened in a system. It represents a distinct, self-standing fact about a system that provides an insight that can be actionable.

Every event has common information like __source__ of the event, __time__ the event took place, and a unique identifier. Event every also has a __type__.

Events published to Event Grid land on a topic.

Event Example:
```
{
    "specversion" : "1.0",
    "type" : "com.yourcompany.order.created",
    "source" : "https://yourcompany.com/orders/",
    "subject" : "O-28964",
    "id" : "A234-1234-1234",
    "time" : "2018-04-05T17:31:00Z",
    "comexampleextension1" : "value",
    "comexampleothervalue" : 5,
    "datacontenttype" : "application/json",
    "data" : {
       "orderId" : "O-28964",
       "URL" : "https://com.yourcompany/orders/O-28964"
    }
}
```

#### Publisher

Application that sends events to Event Grid

#### Namespace

An Event Grid namespace is a management container. With an Azure Event Grid namespace, you can group related resources and manage them as a single unit in your Azure subscription.

A namespace also provides DNS-integrated network endpoints. It also provides a range of access control and network integration management features such as public IP ingress filtering and private links.

#### Throughput units

Define the ingress and egress event rate capacity in namespaces

#### Topic

A topic holds events that have been published to Event Grid. You typically use a topic resource for a collection of related events.

__Namespace topics__ are topics that are created within an Event Grid namespace. Your application publishes events to an HTTP namespace endpoint specifying a namespace topic where published events are logically contained. When designing your application, you have to decide how many topics to create. For relatively large solutions, create a namespace topic for each category of related events. 

Namespace topics support pull delivery and push delivery.

#### Subscriptions

An event subscription is a configuration resource associated with a single topic. Among other things, you use an event subscription to set the event selection criteria to define the event collection available to a subscriber out of the total set of events available in a topic. You can filter events according to the subscriber's requirements. 

For example, you can filter events by their event type. You can also define filter criteria on event data properties if using a JSON object as the value for the data property

#### Pull delivery

With pull delivery, your application connects to Event Grid to read messages using queue-like semantics. As applications connect to Event Grid to consume events, they are in control of the event consumption rate and its timing. Consumer applications can also use private endpoints when connecting to Event Grid to read events using private IP space.

When delivering events using pull delivery, Event Grid includes an array of objects that in turn includes the event and brokerProperties objects.

With push delivery, you define a destination in an event subscription, a webhook, or an Azure service, to which Event Grid sends events.

```
{
    "value": [
        {
            "brokerProperties": {
                "lockToken": "CiYKJDUwNjE4QTFFLUNDODQtNDZBQy1BN0Y4LUE5QkE3NjEwNzQxMxISChDXYS23Z+5Hq754VqQjxywE",
                "deliveryCount": 2
            },
            "event": {
                "specversion": "1.0",
                "id": "A234-1234-1235",
                "source": "/mycontext",
                "time": "2018-04-05T17:31:00Z",
                "type": "com.example.someeventtype",
                "data": "some data"
            }
        },
        {
            "brokerProperties": {
                "lockToken": "CiYKJDUwNjE4QTFFLUNDODQtNDZBQy1BN0Y4LUE5QkE3NjEwNzQxMxISChDLeaL+nRJLNq3/5NXd/T0b",
                "deliveryCount": 1
            },
            "event": {
                "specversion": "1.0",
                "id": "B688-1234-1235",
                "source": "/mycontext",
                "type": "com.example.someeventtype",
                "time": "2018-04-05T17:31:00Z",
                "data": {
                    "somekey" : "value",
                    "someOtherKey" : 9
                }
            }
        }
    ]
}
```

When to use it:
- You need full control as to when to receive events
- You need full control over event consumption
- You want to use private links when receiving events

#### Push delivery

With push delivery, Event Grid sends events to a destination configured in a push (delivery mode in) event subscription. It provides a robust retry logic in case the destination isn't able to receive events.

With pull delivery, subscriber applications connect to Event Grid to consume events.

When to use it:
- avoid constant polling to determine that a system state change has occurred.

#### push vs pull

Pull:
- scenarios requiring low latency, high throughput, and real-time updates.
- Consumers must handle backpressure, where they cannot keep up with incoming events, which may cause congestion, loss, or duplication of events
- Consumers must implement idempotency to process the same event multiple times without side effects; this may increase complexity and cost of the logic.

Pull:
- consumers query the storage for new events
- Consumers control the pace and frequency of fetching events and process them at their own convenience. 
- This model is suitable for scenarios where high availability, resilience, and eventual consistency are needed. 
- producers may experience latency, overhead, and failure points when writing events to the storage. 
- Additionally, consumers may generate unnecessary traffic, waste resources, and miss updates when polling the storage. 
- Consumers must handle concurrency when multiple instances of the same consumer access the same events, which can cause conflicts, inconsistencies, or duplicates.

#### Event Handler

An event handler is an application or Azure service that receives events sent by namespace topics' push delivery mechanism. Event handlers, or sometimes called destinations

##### Webhook event handler

You don't have to host your webhook on Azure, which means that you can use a webhook that's hosted elsewhere to handle events in your application

To protect your webhook from unexpected event delivery, your webhook needs to indicate if it agrees with the event delivery. To that end, your endpoint must handle the webhook validation.

##### EventHub event handler

Several Azure services are automatically configured to handle events and Azure Event Hubs is one of them.

Use Event Hubs when your solution gets events from Event Grid faster than it can process the events. Once the events are in an event hub, your application can process events from the event hub at its own schedule.

### Pros and Cons

### Use cases

#### Build event-driven serverless solutions

Azure Functions -- http --> Event Grid -- http --> Logic App

Use Event Grid to build serverless solutions with Azure Functions Apps, Logic Apps, and API Management. 

#### Receive events from Azure services

Storage Account -- blob created -- http --> Function App

Storage Account -- blob created -- http --> Event Grid  --> WebHook to API deployed on kubernetes

Storage Account -- blob created -- http --> Event Hub

#### Receive events from partner (SaaS providers)

External publisher --http--> Partner Events - Event Grid --http--> Function App


### Examples

#### Creating and Authenticating EventGridPublisherClient

```
EventGridPublisherClient client = new EventGridPublisherClient(
    new Uri(topicEndpoint),
    new AzureKeyCredential(topicAccessKey));
```

#### Publishing Events to Azure Event Grid

Invoke SendEvents or SendEventsAsync to publish the events to Azure Event Grid.

```
// Example of a custom ObjectSerializer used to serialize the event payload to JSON
var myCustomDataSerializer = new JsonObjectSerializer(
    new JsonSerializerOptions()
    {
        PropertyNamingPolicy = JsonNamingPolicy.CamelCase
    });

// Add EventGridEvents to a list to publish to the topic
List<EventGridEvent> eventsList = new List<EventGridEvent>
{
    // EventGridEvent with custom model serialized to JSON
    new EventGridEvent(
        "ExampleEventSubject",
        "Example.EventType",
        "1.0",
        new CustomModel() { A = 5, B = true }),

    // EventGridEvent with custom model serialized to JSON using a custom serializer
    new EventGridEvent(
        "ExampleEventSubject",
        "Example.EventType",
        "1.0",
        myCustomDataSerializer.Serialize(new CustomModel() { A = 5, B = true })),
};

// Send the events
await client.SendEventsAsync(eventsList);
```

#### Consume/subscribe to EventGrid events

```
/**
 * Azure Functions with EventGrid Trigger.
 */
public class EventGridConsumer {
    /**
     * This captures the "Data" portion of an EventGridEvent on a custom topic.
     */
    static class ContosoItemReceivedEventData
    {
        public String itemSku;

        public ContosoItemReceivedEventData(String itemSku) {
            this.itemSku = itemSku;
        }
    }

    /**
     * EventGrid trigger function for handling the events and log them to the execution context.
     */
    @FunctionName("EventGrid-Consumer")
    public void Run(@EventGridTrigger(name = "data") String data, final ExecutionContext executionContext) {
        executionContext.getLogger().info("Java EventGrid trigger function begun\n");
        executionContext.getLogger().info("\tFOUND: " + data);

        try {
            final String StorageBlobCreatedEvent = "Microsoft.Storage.BlobCreated";
            final String CustomTopicEvent = "Contoso.Items.ItemReceived";
            final Gson gson = new GsonBuilder().create();

            EventGridEvent eventGridEvent = gson.fromJson(data, EventGridEvent.class);

            // Deserialize the event data into the appropriate type based on event type
            if (eventGridEvent.eventType().toLowerCase().equals(StorageBlobCreatedEvent.toLowerCase())) {
                // Deserialize the data portion of the event into StorageBlobCreatedEventData
                StorageBlobCreatedEventData eventData = gson.fromJson((String) eventGridEvent.data(), StorageBlobCreatedEventData.class);
                executionContext.getLogger().info("Got BlobCreated event data, blob URI " + eventData.url());
            }
            else if (eventGridEvent.eventType().toLowerCase().equals(CustomTopicEvent.toLowerCase())) {
                // Deserialize the data portion of the event into ContosoItemReceivedEventData
                ContosoItemReceivedEventData eventData = gson.fromJson((String) eventGridEvent.data(), ContosoItemReceivedEventData.class);
                executionContext.getLogger().info("Got ContosoItemReceived event data, item SKU " + eventData.itemSku);
            }
        } catch (Exception e) {
            executionContext.getLogger().info("UNEXPECTED Exception caught: " + e.toString());
        }
    }

}
```

#### Publish events

```
/**
 * Azure Functions with Time Trigger.
 *  - Publish custom topic events which will be captured as EventGrid events
 */
public class EventGridTimeTriggeredCustomPublisher {

    /**
     * This captures the "Data" portion of an EventGridEvent on a custom topic
     */
    static class ContosoItemReceivedEventData
    {
        public String itemSku;

        public ContosoItemReceivedEventData(String itemSku) {
            this.itemSku = itemSku;
        }
    }

    @FunctionName("EventGrid-TimeTriggered-Custom-Publisher")
    public void EventGridWithCustomPublisher(
            @TimerTrigger(name = "timerInfo", schedule = "*/20 * * * * *") String timerInfo,
            final ExecutionContext executionContext) {

        try {
            // Create an event grid client.
            TopicCredentials topicCredentials = new TopicCredentials(System.getenv("EVENTGRID_TOPIC_KEY"));
            EventGridClient client = new EventGridClientImpl(topicCredentials);

            // Publish custom events to the EventGrid.
            System.out.println("Publish custom events to the EventGrid");
            List<EventGridEvent> eventsList = new ArrayList<>();
            for (int i = 0; i < 5; i++) {
                eventsList.add(new EventGridEvent(
                    UUID.randomUUID().toString(),
                    String.format("Door%d", i),
                    new ContosoItemReceivedEventData("Contoso Item SKU #1"),
                    "Contoso.Items.ItemReceived",
                    DateTime.now(),
                    "2.0"
                ));
            }

            String eventGridEndpoint = String.format("https://%s/", new URI(System.getenv("EVENTGRID_TOPIC_ENDPOINT")).getHost());

            client.publishEvents(eventGridEndpoint, eventsList);
        } catch (Exception e) {
            executionContext.getLogger().info("UNEXPECTED Exception caught: " + e.toString());
        }
    }
}
```

## Event Hub

Streaming data, low latency, millions/s, at least once delivery, DATA STREAMING.

Fully managed, real-time data ingestion service that’s simple, trusted, and scalable. Stream millions of events per second from any source.

Integrate seamlessly with other Azure services to unlock valuable insights. Allow existing Apache Kafka clients and applications to talk to Event Hubs without any code changes.

By using streaming data, businesses can gain valuable insights, drive real-time analytics, and respond to events as they happen.

Natively supports Advanced Message Queuing Protocol (AMQP), Apache Kafka, and HTTPS protocols.

Azure Schema Registry in Event Hubs provides a centralized repository for managing schemas of event streaming applications. Schema Registry ensures data compatibility and consistency across event producers and consumers. It enables schema evolution, validation, and governance and promotes efficient data exchange and interoperability.

Event Hubs uses a partitioned consumer model. It enables multiple applications to process the stream concurrently and lets you control the speed of processing. 

Capture your data in near real time in Azure Blob Storage or Azure Data Lake Storage for long-term retention or micro-batch processing.

- __ingestion and storage of large quantities of data__
- __client chosen offsets__
- __data retention__
- streaming data
- fraud detecion
- click streams
- device telemetry

### Concepts

__Producer__ applications: These applications can ingest data to an event hub by using Event Hubs SDKs or any Kafka producer client.

__Namespace__: The management container for one or more event hubs or Kafka topics. The management tasks such as allocating streaming capacity, configuring network security, and enabling geo-disaster recovery are handled at the namespace level.

__Event Hubs__ = __Kafka topic__: In Event Hubs, you can organize events into an event hub or a Kafka topic. It's an append-only distributed log, which can comprise one or more partitions.

__Partitions__: They're used to scale an event hub. If you need more streaming throughput, you can add more partitions.

__Consumer applications:__ These applications can consume data by seeking through the event log and maintaining consumer offset. Consumers can be Kafka consumer clients or Event Hubs SDK clients.

__Consumer group__: This logical group of consumer instances reads data from an event hub or Kafka topic. It enables multiple consumers to read the same streaming data in an event hub independently at their own pace and with their own offsets. A view (state, position, or offset) of an entire event hub. You pass in the same consumer group as a parameter when constructing EventHubConsumerClients, then those clients will be associated with the same consumer group.

| __Kafka Concept__   |  __Event Hubs Concept__
| --------------------|----------------------------|
| Cluster 	          |  Namespace                 |
| Topic 	          |  An event hub
| Partition 	      |  Partition
| Consumer Group 	  |  Consumer Group
| Offset 	          |  Offset

### Integrations

#### Azure Stream Analytics

Event Hubs integrates with Azure Stream Analytics to enable __real-time stream__ processing. With the built-in no-code editor, you can develop a Stream Analytics job by using drag-and-drop functionality, without writing any code. Alternatively, developers can use the SQL-based Stream Analytics query language. 

#### Azure Data Explorer

Azure Data Explorer is a fully managed platform for big data analytics that delivers high performance and allows for the analysis of large volumes of data in near real time. By integrating Event Hubs with Azure Data Explorer, you can perform near real-time analytics and exploration of streaming data.

#### Azure Functions

Event Hubs also integrates with Azure Functions for serverless architectures.


### Pros and Cons

### Use cases

### Examples



## Service Bus

queue or pub-sub, asynchronous message delivery, PULL-based, FIFO, batch/sessions, dead-letter, transaction, routing, at least once delivery, re-try, SEND COMMANDS (do something)

Azure Service Bus is a fully managed enterprise message broker with __message queues__ and __publish-subscribe topics__. 
Service Bus is used to decouple applications and services from each other, providing the following benefits:

* __Load-balancing__ work across competing workers
* Safely routing and transferring data and control across service and application boundaries
* __Coordinating transactional work__ that requires a high-degree of reliability


- __exactly one processing__
- __exclusive ownership of message (queue)__
- __assignment of work with load-aware balancing__
- __load leveling traffic shapes (see size of queue)__
- __topics can route data, queue at the endof topic__
- jobs processing
- decouple applications
- state propagation
- order processing
- queue, topic (pub-sub)

### Pros and Cons

### Use cases

* __Messaging__. Transfer business data, such as sales or purchase orders, journals, or inventory movements.
* __Decouple applications__. Improve reliability and scalability of applications and services. Producer and consumer don't have to be online or readily available at the same time. 
* __Load balancing__. Allow for multiple competing consumers to read from a queue at the same time
* __Transactions__. Allows you to do several operations, all in the scope of an atomic transaction. For example, the following operations can be done in the scope of a transaction.
  * Obtain a message from one queue.
  * Post results of processing to one or more different queues.
  * Move the input message from the original queue.


### Examples

#### Send messages to queue

https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-python-how-to-use-queues?tabs=passwordless

```
## imports
import asyncio
from azure.servicebus.aio import ServiceBusClient
from azure.servicebus import ServiceBusMessage
from azure.identity.aio import DefaultAzureCredential

## constants
FULLY_QUALIFIED_NAMESPACE = "FULLY_QUALIFIED_NAMESPACE"
QUEUE_NAME = "QUEUE_NAME"

credential = DefaultAzureCredential()

## method to send simgle message
async def send_single_message(sender):
    # Create a Service Bus message and send it to the queue
    message = ServiceBusMessage("Single Message")
    await sender.send_messages(message)
    print("Sent a single message")

## Create a Service Bus client and then a queue sender object to send messages.
async def run():
    # create a Service Bus client using the credential
    async with ServiceBusClient(
        fully_qualified_namespace=FULLY_QUALIFIED_NAMESPACE,
        credential=credential,
        logging_enable=True) as servicebus_client:
        # get a Queue Sender object to send messages to the queue
        sender = servicebus_client.get_queue_sender(queue_name=QUEUE_NAME)
        async with sender:
            # send one message
            await send_single_message(sender)
            # send a list of messages
            await send_a_list_of_messages(sender)
            # send a batch of messages
            await send_batch_message(sender)

        # Close credential when no longer needed.
        await credential.close()

```

Run:
```
asyncio.run(run())
print("Done sending messages")
print("-----------------------")
```

#### Receive messages from queue

```
import asyncio

from azure.servicebus.aio import ServiceBusClient
from azure.identity.aio import DefaultAzureCredential

FULLY_QUALIFIED_NAMESPACE = "FULLY_QUALIFIED_NAMESPACE"
QUEUE_NAME = "QUEUE_NAME"

credential = DefaultAzureCredential()

async def run():
    # create a Service Bus client using the connection string
    async with ServiceBusClient(
        fully_qualified_namespace=FULLY_QUALIFIED_NAMESPACE,
        credential=credential,
        logging_enable=True) as servicebus_client:

        async with servicebus_client:
            # get the Queue Receiver object for the queue
            receiver = servicebus_client.get_queue_receiver(queue_name=QUEUE_NAME)
            async with receiver:
                received_msgs = await receiver.receive_messages(max_wait_time=5, max_message_count=20)
                for msg in received_msgs:
                    print("Received: " + str(msg))
                    # complete the message so that the message is removed from the queue
                    await receiver.complete_message(msg)

        # Close credential when no longer needed.
        await credential.close()
```

run:
```
asyncio.run(run())
```

## Logic Apps

Run automated workflows with little to no code. By using the visual designer and selecting from prebuilt operations, you can quickly build a workflow that integrates and manages your apps, data, services, and systems. It uses conditions and switches to determine the next action.

A workflow always starts with a single __trigger__, which specifies the condition to meet before running any subsequent actions in the workflow. Each time the trigger fires, Azure Logic Apps creates and runs a workflow instance. Trigger follows either a polling pattern or a push pattern. 

Following a trigger, an __action__ is any subsequent step that runs some operation in the workflow. Any action can use the outputs from the previous operations, which include the trigger and any other actions.

Use a __connector__ to work with data, events, and resources in other apps, services, systems, and platforms - without writing code. A connector provides one or more prebuilt operations, which you use as steps in your workflow. In a connector, each operation is either a trigger condition that starts a workflow or a subsequent action that performs a specific task, along with properties that you can configure. Connectors are either __built__ (run directly and natively inside Azure Logic Apps) in or __managed__ (deployed, hosted, and managed in Azure by Microsoft). 

Built-in connectors:
- Schedule: Run workflows using custom and advanced schedules.
- Http: Call an HTTP or HTTPS endpoint by using either the HTTP trigger or action
- Request: When a HTTP request is received: Wait for a request from another workflow, app, or service.
- Batch: Batch messages: Trigger a workflow that processes messages in batches. 
- Ftp: Connect to an FTP or FTPS
- Azure BloB Storage: Connect to your Azure Blob Storage

## Pros and Cons

### Use cases

### Examples

For example, you might start the workflow with a SQL Server trigger that checks for new customer data in an SQL database. Following the trigger, your workflow can have a SQL Server action that gets the customer data. Following this SQL Server action, your workflow can use a different action that processes the data, for example, a Data Operations action that creates a CSV table.



## API Management

Azure API Management is a hybrid, multicloud management platform for APIs across all environments. It supports the complete API lifecycle.

Azure API Management helps customers meet these challenges:

* __Abstract backend architecture__ diversity and complexity from API consumers
* __Securely expose services__ hosted on and outside of Azure as APIs
* __Protect__, accelerate, and __observe__ APIs
* Enable __API discovery__ and consumption by internal and external users

Azure API Management is made up of an __API gateway__, a __management plane__, and a __developer portal__.

### API gateway

All requests from client applications first reach the API gateway, which then forwards them to respective backend services. The API gateway acts as a facade to the backend services, allowing API providers to abstract API implementations and evolve backend architecture without impacting API consumers. The gateway enables consistent configuration of routing, security, throttling, caching, and observability.

* Acts as a facade to backend services by accepting API calls and __routing__ them to appropriate backends
* __Verifies__ API keys and other credentials such as JWT tokens and certificates presented with requests
* Enforces usage quotas and __rate limits__
* Optionally __transforms__ requests and responses as specified in policy statements
* If configured, __caches responses__ to improve response latency and minimize the load on backend services
* Emits __logs, metrics, and traces__ for monitoring, reporting, and troubleshooting

### Management plane

Customers interact with the management plane through: Azure portal, Azure PowerShell, Azure CLI, a Visual Studio Code extension, a REST API, or client SDKs.

Use the management plane to:

* __Provision and configure__ API Management service settings
* Define or import __API schemas__ from a wide range of sources, including OpenAPI, WSDL, and OData definitions, Azure compute services, and WebSocket, GraphQL, and gRPC backends
* Set up policies like __quotas__ or __transformations__ on the APIs
* Get insights from __analytics__
* __Manage users__

#### Add an API manually

* Navigate to your API Management service in the Azure portal and select APIs from the menu.
* From the left menu, select + Add API.
* Select HTTP from the list.
* Enter the backend Web service URL 

#### Subscriptions in Azure API Management

Subscriptions are the most common way for API consumers to access APIs published through an API Management instance

Developers who need to consume the published APIs must include a valid subscription key in HTTP requests.

### Developer portal

Fully customizable website with the documentation of your APIs.

Using the developer portal, developers can:

* Read __API documentation__
* __Call an API__ via the interactive console
* Create an account and __subscribe to get API keys__
* Access __analytics__ on their own usage
* __Download API__ definitions
* __Manage API keys__


## Pros and Cons

### Use cases

### Examples

## Protocols

### MQTT

MQTT is a communication protocol with features specifically targeted at IoT solutions:

- Uses TCP connections, for reliability (assured delivery and packet error checking), fragmentation and ordering.
- Aims to minimize data overhead of each MQTT packet.
- Bi-directional message flow - data from and commands to devices can use the same TCP connection.
- uses SSL/TLS for security
- transmitting information between low-power sensors. 
- It is based on the publish/subscribe model (pub-sub).
- The MQTT protocol can keep a connection open for as long as possible, sending only a single data packet. Unlike HTTP communication, which requires you to open and close a connection (including TCP) for every data packet you want to send

## Cloud Events

A specification for describing event data in a common way

https://cloudevents.io/

https://github.com/cloudevents/spec

## Issues with event-driven

- Complexity
- Debugging: difficult to trace events
- Error handling: root cause analysis
- Security: each new service might be security attack
