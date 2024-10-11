# Table of Contents
- [Event-driven architecture in AWS](#EventDrivenAws)
  - [SQS](#SQS)
  - [SNS](#SNS)

# Event-driven architecture on aws <a id="EventDrivenAws"></a>

__design__: event storming -> ( command (play video), query, event (video uploaded) ) -> services -> add communication patterns (queue, pub-sub (kafka, event-bridge), streaming) -> workflows (step functions)

__pros__: decoupled, scalaility, extensibility,  

__issues__ with data in event: need more data (enrich), sensitive data (encrypt), large messages, 

__event schema__

__cross-domain events__ (boundaries)

__standards__: timestamp, corelationId, domain name, 

__discoverability__: schema registry, 

## SQS  <a id="SQS"></a>

Amazon Simple Queue Service (Amazon SQS) offers a secure, durable, and available hosted queue that lets you integrate and decouple distributed software systems and components.

Benefits:
- __Security__ – You control who can send messages to and receive messages from an Amazon SQS queue. 
You can encrypt Amazon SQS messages with managed server-side encryption (SSE), or by using custom SSE keys managed in AWS Key Management Service (AWS KMS).
- __Durability__ – For the safety of your messages, Amazon SQS stores them on multiple servers. Standard queues support at-least-once message delivery, and FIFO queues support exactly-once message processing and high-throughput mode.
- __Availability__ – Amazon SQS uses redundant infrastructure to provide highly-concurrent access to messages and high availability for producing and consuming messages.
- __Scalability__ – Amazon SQS can process each buffered request independently, scaling transparently to handle any load increases or spikes without any provisioning instructions.
- __Reliability__ – Amazon SQS locks your messages during processing, so that multiple producers can send and multiple consumers can receive messages at the same time.

Queue Types:
- standard
  - Unlimited throughput: very high, nearly unlimited number of API calls per second
  - At-least-once delivery
  - Best-effort ordering
  - Durability and redundancy – Standard queues ensure high durability by storing multiple copies of each message across multiple AWS Availability Zones. 
  - Visibility timeout – Amazon SQS allows you to configure a visibility timeout to control how long a message stays hidden after being received, ensuring that other consumers do not process the message until it has been fully handled or the timeout expires.
- FIFO
  - High throughput – When you use batching, FIFO queues process up to 3,000 messages per second per API method ( 300 API calls/s * batch of 10 message)
  - Exactly-once processing – FIFO queues deliver each message once and keep it available until you process and delete it.
  - First-in-first-out delivery – FIFO queues ensure that you receive messages in the order
  - Key terms:
    - Message deduplication ID: A token used in Amazon SQS FIFO queues to uniquely identify messages and prevent duplication. If multiple messages with the same deduplication ID are sent within a 5 minute deduplication interval, they are treated as duplicates, and only one copy is delivered. If you don't specify a deduplication ID and content-based deduplication is enabled, Amazon SQS generates a deduplication ID by hashing the message body.
    - Message group ID: The tag that specifies that a message belongs to a specific message group. Messages that belong to the same message group are always processed one by one, in a strict order relative to the message group
    - Sequence number: The large, non-consecutive number that Amazon SQS assigns to each message
    - Sending messages: messages are ordered based on message group ID. If multiple hosts (or different threads on the same host) send messages with the same message group ID to a FIFO queue, Amazon SQS stores the messages in the order in which they arrive for processing.
    - Receiving a message: You can't request to receive messages with a specific message group ID. When receiving messages from a FIFO queue with multiple message group ID
    - FIFO queues allow the producer or consumer to attempt multiple retries: SendMessage or ReceiveMessage

Dead Letter queues: 
- Amazon SQS supports dead-letter queues (DLQs), which source queues can target for messages that are not processed successfully. 
- For optimal performance, it is a best practice to keep the source queue and DLQ within the same AWS account and Region
- You can Move messages out of the dead-letter queue using dead-letter queue redrive.
- Amazon SQS does not create the dead-letter queue automatically. You must first create the queue before using it as a dead-letter queue. 
- The dead-letter queue of a FIFO queue must also be a FIFO queue. Similarly, the dead-letter queue of a standard queue must also be a standard queue.
- Use a redrive policy to specify the maxReceiveCount. The maxReceiveCount is the number of times a consumer can receive a message from a source queue before it is moved to a dead-letter queue. For example, if the maxReceiveCount is set to a low value such as 1, one failure to receive a message would cause the message to move to the dead-letter queue.
- When a message is moved to a dead-letter queue, the enqueue timestamp is unchanged

SQS short and long polling:
- Short polling (default) – The ReceiveMessage request queries a subset of servers (based on a weighted random distribution) to find available messages and sends an immediate response, even if no messages are found.
- Long polling – ReceiveMessage queries all servers for messages, sending a response once at least one message is available, up to the specified maximum. An empty response is sent only if the polling wait time expires. This option can reduce the number of empty responses and potentially lower costs.
  Long polling helps reduce the cost of using Amazon SQS by eliminating the number of empty responses (when there are no messages available for a ReceiveMessage request) and false empty responses (when messages are available but aren't included in a response).

Short polling occurs when the WaitTimeSeconds parameter of a ReceiveMessage request is set to 0 in one of two ways:
- The ReceiveMessage call sets WaitTimeSeconds to 0.
- The ReceiveMessage call doesn’t set WaitTimeSeconds, but the queue attribute ReceiveMessageWaitTimeSeconds is set to 0.

SQS visibility timeout:
- When a consumer receives a message from an Amazon SQS queue, the message remains in the queue but becomes temporarily invisible to other consumers. This temporary invisibility is controlled by the visibility timeout, a mechanism that prevents other consumers from processing the same message while it is being worked on. Amazon SQS does not automatically delete the message; instead, the consumer must explicitly delete the message using the DeleteMessage action after it has been successfully processed
- If the consumer fails to delete the message before the visibility timeout expires, the message becomes visible again in the queue and can be retrieved by another consumer.
- SQS queue has a default visibility timeout of 30 seconds,

In flight messages:
- in-flight messages are those that have been received from a queue by a consumer but have not yet been deleted.
- To manage in-flight messages effectively:
  - Prompt deletion – Ensure that messages are deleted as soon as they are successfully processed
  - Monitor with CloudWatch – Use Amazon CloudWatch to monitor the number of in-flight messages
  - Distribute load – Consider increasing the number of queues if you are processing a high volume of messages

# SNS <a id="SNS"></a>

