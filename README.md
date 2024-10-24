# EventPeople

**EventPeople** is an open-source, technology-agnostic framework designed to simplify communication in event-based services, based on the [EventBus](https://github.com/EmpregoLigado/event_bus_rb) gem. By leveraging a structured event naming convention and robust event handling mechanisms, EventPeople facilitates scalable and maintainable interactions between services sharing a queue.

## Table of Contents

1. [Introduction](#introduction)
2. [Event Naming Convention](#event-naming-convention)
3. [Architecture Overview](#architecture-overview)
4. [Class Structure and Responsibilities](#class-structure-and-responsibilities)
    - [Core Components](#core-components)
    - [Retry and Dead Letter Queue (DLQ) Components](#retry-and-dead-letter-queue-dlq-components)
5. [Behavior and Workflow](#behavior-and-workflow)
    - [Emitting Events](#1-emitting-events)
    - [Consuming Events](#2-consuming-events)
    - [Running as a Daemon](#3-running-as-a-daemon)
    - [Retry and Dead Letter Queue (DLQ) Processing](#4-retry-and-dead-letter-queue-dlq-processing)
6. [Usage Scenarios](#usage-scenarios)
7. [Implementation Guidelines](#implementation-guidelines)
8. [Future Enhancements](#future-enhancements)
9. [License](#license)

---

## Introduction

EventPeople is designed to streamline the communication of event-based services across different systems. By abstracting the complexities of event handling and providing a clear structure for event naming and consumption, EventPeople enables developers to build scalable and maintainable applications with ease.

**Key Benefits:**

- **Structured Communication:** Clear event naming conventions enhance clarity and reduce ambiguity.
- **Flexible Consumption:** Subscribe to specific events or patterns, enabling targeted and efficient event handling.
- **Scalable Architecture:** Designed to handle large volumes of events seamlessly.
- **Extensible Design:** Easily adaptable to support various message brokers and integration needs.
- **Resilient Processing:** Built-in retry mechanisms and Dead Letter Queues ensure reliable event processing.

---

## Event Naming Convention

A consistent event naming strategy is crucial for clarity and effective event routing. EventPeople adopts a four-part naming convention:

```
resource.origin.action.destination
```

### Components:

1. **Resource:** The entity related to the event (e.g., `user`, `product`, `order`).
2. **Origin:** The system or service emitting the event (e.g., `auth_service`, `inventory_system`).
3. **Action:** The operation performed on the resource, preferably in simple past tense (e.g., `created`, `updated`, `deleted`).
4. **Destination** *(Optional):* Specifies the target service for the event. If omitted, defaults to `all`, indicating that all services can consume the event.

### Examples:

- `user.auth_service.created.all`  
  *A user creation event emitted by `auth_service` intended for all services.*

- `order.order_service.updated.payment_service`  
  *An order update event from `order_service` targeted at `payment_service`.*

---

## Architecture Overview

![event_people class' diagram](diagram.png "EventPeople Class' Diagram")

**Core Components:**

- **Emitter:** Emits events into the broker.
- **Listener:** Subscribes to events from the broker based on naming patterns.
- **Daemon:** Runs as a background process to manage continuous event consumption.
- **Broker:** Manages the transmission and routing of events (currently RabbitMQ).
- **Retry Mechanism:** Automatically retries failed event processing.
- **Dead Letter Queue (DLQ):** Archive events that failed to process after retries.

---

## Class Structure and Responsibilities

The following outlines the key classes/interfaces and their roles within EventPeople.

### Core Components

#### 1. BaseBroker (Interface)

**Responsibilities:**

- Manage connections to the message broker.
- Handle event consumption and production.

**Methods:**

- `getConsumers()`: Retrieve active consumers.
- `getConnection()`: Obtain the current broker connection.
- `consume(eventName, callback)`: Subscribe to an event with a callback.
- `produce(events)`: Emit events to the broker.
- `closeConnection()`: Terminate the broker connection.

#### 2. Config (Static Class)

**Responsibilities:**

- Store and provide configuration settings for EventPeople, including default retry and DLQ behaviors.

**Attributes:**

- `broker`: The message broker in use (e.g., RabbitMQ).
- `appName`: Name of the application emitting events.
- `topicName`: The topic or exchange name in the broker.
- `vhostName`: Virtual host name for broker connection.
- `URL`: Connection URL for the broker.
- `fullURL`: Complete connection string.
- `maxAttempts`: Number of the default max attempts for each event.
- `delayStrategy`: The default retry delaying strategy. Can be `fixed` or `exponential`.
- `dlqName`: Name of the default DLQ, if not defined it defaults to `<queue_name>_dlq`. It is only recommended setting this value if you want to concentrate all dead jobs in the same DLQ.

**Methods:**

- `getBroker()`: Retrieve broker configuration.
- `getRetryConfig()`: Retrieves default configurations for retries and DLQs.

#### 3. Event

**Responsibilities:**

- Represent an event with its metadata and payload.

**Attributes:**

- `name`: Event name following the naming convention.
- `headers`: Metadata associated with the event.
- `body`: The main content or data of the event.
- `schemaVersion`: Version of the event schema.
- `retryCount`: Number of times the event has been retried (automatically set to 0 on initialization).

**Methods:**

- `payload()`: Retrieve the event payload.
- `hasBody()`: Check if the event has a payload.
- `hasName()`: Validate the presence of an event name.
- `buildPayload()`: Construct the payload.
- `generateHeaders()`: Create event headers.
- `fixName()`: Adjust the event name to adhere to conventions.
- `incrementRetryCount()`: Increase the retry count by one.

> **Note:** `retryCount` is automatically initialized to 0 when an `Event` instance is created.

#### 4. Listener (Static Class)

**Responsibilities:**

- Facilitate subscription to events with optional retry and DLQ configurations.

**Methods:**

- `on(eventName, callback, maxAttempts = null, delayStrategy = null, dlqName = null)`:  
  Subscribe to a specific event or pattern with a callback function, including optional parameters for retry and DLQ configurations. If parameters are not provided, default values from `Config` are used internally.

> **Note:** The `fail()` method within the `Context` interface handles retry and DLQ logic internally. Users do not need to implement retry mechanisms manually.

#### 5. Emitter (Static Class)

**Responsibilities:**

- Handle the emission of events.

**Methods:**

- `trigger(events)`: Emit one or multiple events to the broker.

#### 6. Daemon (Static Class)

**Responsibilities:**

- Manage background processes for event consumption.

**Methods:**

- `start()`: Initiate the daemon process.
- `stop()`: Terminate the daemon process.
- `bindSignals()`: Handle system signals for graceful shutdowns.

#### 7. Context (Interface)

**Responsibilities:**

- Provide context to the callback processing the event, it also has methods to handle the outcome of event processing.

**Attributes:**

- `maxRetries`: Total number of retries for the event being processed.
- `isLastRetry`: Informs if this is the last attempt for the event being processed, useful to customize behavior on the event's last retry.

**Methods:**

- `success()`: Indicate successful processing of an event.
- `fail()`: Indicate failed processing, activating retry and DLQ rules.
- `reject()`: Indicate rejection of an event, routing it directly to the DLQ without retrying.

> **Note:** The `fail()` method automatically handles retry and DLQ logic internally.

#### 8. Listeners (Package)

##### a. BaseListener

**Responsibilities:**

- Serve as a base class for specific listeners.

**Attributes:**

- `context`: The processing context for event outcomes.

**Methods:**

- `callback(function, event, maxAttempts = null, delayStrategy = null, dlqName = null)`: Default callback to receive and encapsulate the return from the Broker and then prepare and send the actual event data to the user's defined callback.
- `bindEvent(function, event_name)`: Bind a function to a specific event name.
- `success()`: Mark event processing as successful.
- `fail()`: Mark event processing as failed, triggering internal retry and DLQ handling.
- `reject()`: Reject the event, sending it directly to the DLQ if enabled.
- `fixed_event_name(event_name, postfix)`: Adjust event name with the `.all` postfix if its missing.

##### b. ListenersManager (Static Class)

**Responsibilities:**

- Manage all active listeners and their configurations.

**Attributes:**

- `listeners`: Collection of the configurations of all active listeners.

**Methods:**

- `bindAllListeners()`: Bind all defined listeners.
- `addListener(configuration)`: Add a new listener to the manager based on a HashMap with its configurations.

#### 9. Rabbit (Package)

The implementation in RabbitMQ relies on a central Topic which routes the events to the queues based on the event name (routing key). When a Listener is bound to an event name, the lib creates the queue if not available and binds it to the Topic, based on both specific (with the `.service_name` suffix) and the generic (with the `.all` suffix) routing keys.

For more information about RabbitMQ concept check in this [link](https://www.rabbitmq.com/tutorials/amqp-concepts).

##### a. Topic

**Responsibilities:**

- Manage the main Topic within RabbitMQ.

**Attributes:**

- `channel`: The communication channel with RabbitMQ.
- `topic`: The specific topic or exchange name.

**Methods:**

- `getTopic()`: Retrieve the topic name.
- `getChannel()`: Retrieve the communication channel.
- `produce(event)`: Emit an event to the topic.
- `topicOptions()`: Define options for the topic.

##### b. Queue

**Responsibilities:**

- Manage queues within RabbitMQ.

**Attributes:**

- `channel`: The communication channel with RabbitMQ.
- `topic`: The topic associated with the queue.

**Methods:**

- `subscribe(routingKey, callback)`: Subscribe to a routing key with a callback.
- `callback(deliverInfo, properties, payload, function)`: Handle incoming messages.
- `queueOptions()`: Define options for the queue.
- `queueName(routingKey)`: Generate queue name based on routing key.

##### c. RabbitContext (Implements Context)

**Responsibilities:**

- Handle the context of event processing within RabbitMQ.

**Attributes:**

- `channel`: The communication channel.
- `delivery_info`: Information about the delivery.
- `maxRetries`: Maximum number of retries allowed for the event.
- `isLastRetry`: Boolean indicating if the current attempt is the last retry.
- `dlqName`: Name of the DLQ to be used in the actual context.

**Methods:**

- `success()`: Acknowledge successful processing.
- `fail()`: Requeue the message upon failure, activating retry and DLQ rules.
- `reject()`: Discard the message without requeueing, routing it directly to the DLQ.

##### d. RabbitBroker (Implements BaseBroker)

**Responsibilities:**

- Implement broker functionalities specific to RabbitMQ.

**Attributes:**

- `connection`: The connection instance with RabbitMQ.
- `consumers`: Active consumers.
- `channel`: Communication channel.
- `session`: Session details.
- `topic`: Associated topic.
- `queue`: Associated queue.
- `rabbitURL`: Connection URL for RabbitMQ.

**Methods:**

- `getConsumers()`: Retrieve active consumers.
- `getConnection()`: Retrieve the current connection.
- `consume(eventName, function)`: Subscribe to an event with a callback.
- `produce(events)`: Emit events to the broker.
- `closeConnection()`: Terminate the broker connection.

---

### Retry and Dead Letter Queue (DLQ) Components

To support robust retry mechanisms and DLQ functionality internally, the following additional classes and packages are proposed. These components are managed within the framework and are not exposed directly to the external API.

#### 10. RetryManager (Internal Static Class)

**Responsibilities:**

- Manage retry and DLQ policies as well as execution for failed event processing.

**Attributes:**

- `maxAttempts`: Maximum number of retry attempts.
- `initialDelay`: Initial delay before the first retry (in milliseconds).
- `maxDelay`: Maximum delay between retries (in milliseconds).
- `currentAttempt`: Counter for current retry attempt.
- `dlqName`: Name of the Dead Letter Queue.

**Methods:**

- `shouldRetry()`: Determine if the event should be retried based on `currentAttempt` and `maxAttempts`.
- `getNextDelay()`: Calculate the next delay to be used in exponential backoff.
- `incrementAttempt()`: Increase the retry count by one.
- `reset()`: Reset the retry counter for new events.
- `moveToDLQ(event)`: Move the failed event to the DLQ.

---

## Behavior and Workflow

### 1. Emitting Events

- **Process:**
  1. An event is instantiated with a name, payload.
  2. The `Emitter` triggers the event, sending it to the broker.
  3. The broker routes the event based on its naming convention to the appropriate queues.

- **Outcome:**
  - Subscribed listeners receive the event and execute their callback functions.

### 2. Consuming Events

- **Process:**
  1. A `Listener` subscribes to specific event names or patterns.
  2. Upon receiving an event, the listener processes it using the defined callback.
  3. The listener invokes one of the context methods (`success`, `fail`, `reject`) based on the processing outcome.

- **Outcome:**
  - **Success:** Event is acknowledged and removed from the queue.
  - **Fail:** Event is requeued for retry, activating retry and DLQ rules.
  - **Reject:** Event is discarded and routed directly to the DLQ without retrying.

### 3. Running as a Daemon

- **Process:**
  1. The `Daemon` starts and initializes all event bindings.
  2. It continuously listens for incoming events and delegates them to the appropriate listeners.
  3. Handles graceful shutdowns and signal bindings.

- **Outcome:**
  - Background processes efficiently manage event consumption without manual intervention.

### 4. Retry and Dead Letter Queue (DLQ) Processing

- **Process:**
  1. When an event processing fails (i.e., `fail()` is invoked), the system detects the failure and initiates a retry based on the retry policy.
  2. The `RetryManager` calculates the next retry delay using an exponential backoff strategy.
  3. The event is requeued with an updated `retryCount`.
  4. If the event continues to fail after exhausting all retry attempts, the `Listener` using the `RetryManager` moves the event to the Dead Letter Queue (DLQ) for further inspection or manual intervention.

- **Outcome:**
  - Ensures that transient issues are handled gracefully through retries.
  - Prevents permanently failing events from clogging the main queues by moving them to the DLQ.

---

## Usage Scenarios

### 1. User Management System

- **Scenario:**
  - When a new user is created (`user.auth_service.created.all`), the `UserService`, `EmailService`, and `AnalyticsService` consume the event to perform respective actions.

### 2. E-commerce Platform

- **Scenario:**
  - An order update event (`order.order_service.updated.inventory_service`) is emitted when an order's status changes, allowing the `InventoryService` to adjust stock levels accordingly.

### 3. Notification System

- **Scenario:**
  - A `notificationService` listens to various events to send real-time notifications to users based on their interactions across different services.

---

## Implementation Guidelines

### 1. Setting Up the Broker

- **Connection:**
  - Establish a connection to the chosen message broker (e.g., RabbitMQ).
  - Ensure proper configuration parameters are set (e.g., URL, vhost).

- **Topic and Queue Management:**
  - Define the topic based on event naming conventions.
  - Set up queues with appropriate routing keys to facilitate event routing.
  - **Important:** The library **does not** create virtual hosts (`vhost`) or topics to avoid duplication or confusion, given the lib relies on a central Topic to route the events to the correct queues. Ensure that these are pre-configured correctly in the broker.

### 2. Defining Events

- **Event Creation:**
  - Instantiate events with names adhering to the full (`resource.origin.action.destination`) or the reduced (`resource.origin.action`) formats.
  - Ensure payloads are structured as hash/dictionary objects for consistency.
  - `retryCount` is automatically initialized to 0 for new events.

- **Validation:**
  - Implement validation to ensure event names and payloads meet the required standards.

### 3. Emitting Events

- **Emitter Integration:**
  - Utilize the `Emitter` component to trigger events.
  - Handle acknowledgments and retries as necessary based on broker feedback.

### 4. Subscribing to Events

- **Listener Configuration:**
  - Use the `Listener` to subscribe to specific events or patterns using wildcards (`*`, `#`).
  - Define callback functions to process incoming events.

- **Wildcard Patterns:**
  - `*` (star): Matches exactly one word.
    - Example: `user.*.created.all` matches `user.auth_service.created.all`, `user.user_service.created.all`.
  - `#` (hash): Matches zero or more words.
    - Example: `user.#.all` matches `user.auth_service.created.all`, `user.account_service.updated.all`, etc.

### 5. Handling Event Outcomes

- **Context Methods:**
  - Within callback functions, invoke `success()`, `fail()`, or `reject()` based on processing results to inform the broker of the event's status.
  - **`success()`:** Inform the event has been successfuly processed.
  - - **`fail()`:** Activates retry and DLQ rules.
  - **`reject()`:** Routes the event directly to the DLQ without retrying.

### 6. Running as a Daemon

- **Daemon Initialization:**
  - Initialize the `Daemon` to manage continuous event consumption.
  - Ensure all event bindings are defined before starting the daemon.

- **Graceful Shutdown:**
  - Implement signal handling to allow the daemon to shut down gracefully, ensuring no events are lost or left unprocessed.

### 7. Extensibility

- **Adding New Brokers:**
  - Implement the `BaseBroker` interface and any other necessary structure for additional message brokers as needed.
  - Ensure new brokers adhere to the event handling and routing conventions defined by EventPeople as well as they encapsulate all the Broker logic, providing a clean interface to the users.

### 8. Retry and Dead Letter Queue (DLQ) Logic

Implementing robust retry and DLQ mechanisms is vital for ensuring that transient failures do not result in lost events and that persistent issues are appropriately isolated for further investigation.

#### A. Default Configuration

To streamline implementation, provide sensible default settings that can be overridden as needed. These defaults are configured within the `Config` class and are applied globally unless overridden per listener.

- **Retry Attempts:** Default to 3 retries.
- **Retry Delay Strategy:** Implement exponential backoff and fixed strategies.
- **DLQ Naming Convention:** Use the original queue name appended with `_dlq` as a suffix.
- **Max Retry Delay:** Default to 10 minutes (600,000 milliseconds) to prevent excessively long wait times.

**Default Exponential Backoff Attributes:**

- **Initial Delay:** 1 second (1000 milliseconds)
- **Multiplier:** 5 (delay grows 5 times with each retry)
- **Max Delay:** 10 minutes (600,000 milliseconds)

#### B. Implementing Retry Logic

1. **Failure Detection:**
   - When an event processing fails (i.e., `fail()` is invoked), the system detects the failure and initiates a retry based on the retry policy.
   - If the user's callback raises an error, the `Listener` must catch it and call the `fail()` method automatically.

2. **Retry Policy:**
   - **Exponential Backoff:** Increase the delay between each retry attempt exponentially to reduce load during transient failures.
   - **Retry Counter:** Track the number of retry attempts to prevent infinite retry loops.

3. **Retry Mechanism:**
   - Requeue the event with an updated `retryCount`.
   - Ensure that each retry attempt increments the `retryCount`.

4. **Max Retry Limit:**
   - After exceeding the maximum number of retry attempts, the event should be moved to the DLQ for manual inspection or alternative processing.

#### C. Dead Letter Queue (DLQ) Implementation

1. **DLQ Setup:**
   - The library automatically creates DLQs and handles the routing of messages that have exceeded their retry attempts.
   - Instead of relying on RabbitMQ's built-in DLQ strategy, EventPeople implements its own mechanism by routing messages to a designated DLQ queue, which is a regular queue used as a DLQ.
   - The DLQ is created automatically by the library, following a standardized naming convention (e.g., `primary_queue_dlq`).

2. **Processing DLQ Events:**
   - Since the DLQ is a regular queue, messages in the DLQ are not automatically reprocessed.
   - To reprocess dead messages, you need to manually move them back to the original queue.
   - Implement mechanisms or administrative tools to monitor and manage events in the DLQ, including reprocessing or discarding them as necessary.

3. **Logging and Alerts:**
   - The library logs all events moved to the DLQ with detailed error information.
   - It's recommended to set up alerting systems to notify relevant stakeholders when events are sent to the DLQ, enabling prompt investigation.

#### D. Simplified Listener Interface for Retry and DLQ

To simplify the integration of retry and DLQ functionalities, **EventPeople** allows listeners to receive only essential parameters during event binding. The framework internally manages the instantiation and configuration of retry and DLQ components based on these parameters. If parameters are not provided, default configurations from the `Config` class are applied.

**Listener Binding with Retry and DLQ:**

```pseudo
// Applying Retry and DLQ to a Listener with Custom Configurations
Listener.on(
    "order.order_service.updated.inventory_service",
    callbackFunction,
    maxAttempts = 5,
    delayStrategy = "exponential", // Options: "fixed", "exponential"
    dlqName = "order_service_dlq"
)
```

**Interface Parameters:**

- **`maxAttempts` (Integer):** Maximum number of retry attempts. Defaults to `Config.defaultRetryConfig.maxAttempts` if not provided.
- **`delayStrategy` (String):** Strategy for delay between retries. Available options:
  - `"fixed"`: Fixed delay between retries.
  - `"exponential"`: Exponential backoff starting at 1 second (default).
- **`dlqName` (String):** Name of the Dead Letter Queue. Defaults to the original queue name with `_dlq` suffix if not provided.

**Default Exponential Backoff Configuration:**

- **Initial Delay:** 1 second (1000 milliseconds)
- **Multiplier:** 5 (delay  grows 5 times with each retry)
- **Max Delay:** 10 minutes (600,000 milliseconds)

**Usage Example:**

```pseudo
// Define callback
function callbackFunction(event, context) {
    // Process event
    if (createUser(event.payload)) {
        context.success()
    } else {
        context.fail()
    }
}

// Subscribe to an event
Listener.on(
    "user.auth_service.created.all",
    callbackFunction,
    maxAttempts = 5,
    delayStrategy = "exponential",
    dlqName = "user_service_dlq"
)
```

**Handling Retry Count and `isLastRetry` within Listeners:**

While the retry logic is handled internally, listeners can still access the `retryCount` and `isLastRetry` attributes within the `Event` object to make informed decisions during event processing.

```pseudo
// Listening to an Event with Access to retryCount and isLastRetry
Listener.on(
    "user.auth_service.created.all",
    function(event, context) {
        // Optional: Customize behavior based on retry count
        if (event.retryCount > 0) {
            log("Retry attempt #" + event.retryCount + " for event: " + event.name)
        }
    
        // Process event
        createUser(event.payload)
        context.success()
    },
    maxAttempts = 3,
    delayStrategy = "exponential", // "fixed" or "exponential"
    dlqName = "user_service_dlq"
)
```

**Example with `isLastRetry`:**

```pseudo
// Listening to an Event with Access to maxRetries and isLastRetry
Listener.on(
    "order.order_service.updated.inventory_service",
    function(event, context) {
        try {
            // Optional: Log retry attempts
            if (event.retryCount > 0) {
                log("Retry attempt #" + event.retryCount + " for event: " + event.name)
            }

            // Process event
            updateInventory(event.payload)
            context.success()
        } catch (error) {
            // Customize behavior on the last retry
            if (event.isLastRetry) {
                log("Final retry attempt reached for event: " + event.name)
                // Perform additional actions if needed before failing
            }

            raise error // Whenever an error is raised within the callback the Listener calls fail() automatically
        }
    },
    maxAttempts = 5,
    delayStrategy = "exponential", // "fixed" or "exponential"
    dlqName = "inventory_service_dlq"
)
```

**Default Configuration Handling:**

If `maxAttempts`, `delayStrategy`, or `dlqName` are not provided during listener binding, **EventPeople** automatically applies the default configurations defined in the `Config` class.

```pseudo
// Listener Binding without Custom Configurations (Defaults Applied)
Listener.on(
    "product.inventory_system.updated.#",
    function(event, context) {
        // Process event
        updateInventory(event.payload)
        context.success()
    }
)
```

In this example, since `maxAttempts`, `delayStrategy`, and `dlqName` are not provided, **EventPeople** utilizes the default settings from the `Config` class. The DLQ name defaults to the original queue name with a `_dlq` suffix.

---

## Future Enhancements

- **Support for Additional Message Brokers:**
  - Integrate with brokers like Kafka, AWS SNS/SQS, or Google Pub/Sub to provide flexibility in deployment environments.

- **Schema Validation:**
  - Incorporate schema validation to ensure payloads adhere to predefined structures, enhancing data integrity and reliability as well as adding different callbacks to different schema versions, improving the maintanability of the systems.

---

## Implementations and Contributions

We have the tool implemented in 4 Technologies:

- Ruby: https://github.com/pin-people/event_people_ruby
- Python: https://github.com/pin-people/event_people_python
- Node(Typescript): https://github.com/pin-people/event_people_node
- Go: https://github.com/pin-people/event_people_go

But if you want to use it on your project and the Technology is not supported yet, don't worry, you can use the guidelines in this repository to implement it!

---

## License

The module is available as open source under the terms of the [LGPL 3.0 License](https://www.gnu.org/licenses/lgpl-3.0.en.html).
