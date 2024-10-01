# Event-driven architecture on azure

## Event Grid

Scalable, fully managed Pub Sub message distribution service that offers flexible message consumption patterns using the MQTT and HTTP protocols.
You can build data pipelines with device data, integrate applications, and build event-driven serverless architectures.
Event Grid can be configured to send events to subscribers (push delivery) or subscribers can connect to Event Grid to read events (pull delivery).

DISCRETE FACTS, APPLICATIONS REACT TO EVENTS (automate business process and workflow, react to status change, integrate with external service)

serverless, service-to-service communication, dynamically scalable, at least once delivery, schema, keeps trying to deliver for 24 hours, PUSH or PULL-based.

Publish-subscribe messaging model - Communicate efficiently using one-to-many, many-to-one, and one-to-one messaging patterns.

### Featues

__Push__: push delivery mechanism sends data to destinations that include your own application webhooks and Azure services. Push delivery features a 24-hour retry mechanism with exponential backoff to make sure events are delivered.

Your own __application events__ - Use Event Grid to route, filter, and reliably deliver custom events from your app.

__Partner events __– Subscribe to your partner SaaS provider events and process them on Azure.

__Advanced filtering__ – Filter on event type or other event attributes to make sure your event handlers or consumer apps receive only relevant events.

__Source of data__: 
* MQTT clients’ data
* HTTP: Azure services, Your custom applications, External partner (SaaS) systems

__High throughput __- Build high-volume integrated solutions with Event Grid.

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

#### Event Handler

An event handler is an application or Azure service that receives events sent by namespace topics' push delivery mechanism. Event handlers, or sometimes called destinations

##### Webhook event handler

You don't have to host your webhook on Azure, which means that you can use a webhook that's hosted elsewhere to handle events in your application

To protect your webhook from unexpected event delivery, your webhook needs to indicate if it agrees with the event delivery. To that end, your endpoint must handle the webhook validation.




### Pros and Cons

### Use cases

#### Build event-driven serverless solutions

Azure Functions -- http --> Event Grid -- http --> Logic App

Use Event Grid to build serverless solutions with Azure Functions Apps, Logic Apps, and API Management. 

#### Receive events from Azure services
							--> Function App
Storage Account -- blob created -- http --> Event Grid  --> WebHook to API deployed on kubernetes
							--> Event Hub

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

streaming data, low latency, millions/s, at least once delivery, DATA STREAMING

## Pros and Cons

### Use cases

### Examples



## Service Bus

queue or pub-sub, asynchronous message delivery, PULL-based, FIFO, batch/sessions, dead-letter, transaction, routing, at least once delivery, re-try, SEND COMMANDS (do something)

## Pros and Cons

### Use cases

### Examples



## Logic Apps

## Pros and Cons

### Use cases

### Examples



## API Management

## Pros and Cons

### Use cases

### Examples