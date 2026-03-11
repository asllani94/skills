---
name: baker-framework
description: >
  Expert skill for Baker — an ING-internal framework for orchestrating microservice-based
  process flows using a declarative recipe DSL. Use this skill when working in any ING
  Spring Boot / Java 21 project that integrates with Baker, when defining business process
  orchestration, creating recipes with interactions and events, handling errors in distributed
  workflows, or visualizing process flows. Covers recipe DSL (Java, Kotlin, Scala), interactions,
  ingredients, events, error handling strategies, testing with baker-test library, runtime
  configuration, and verbatim code examples.
---

# Baker Framework — Complete Knowledge Base

## Table of Contents
1. Overview
2. Core Concepts
3. Configuration Reference
4. Code Examples
5. Integration with Other ING Tools
6. Pitfalls & Anti-patterns
7. FAQ
8. Glossary

---

## 1. Overview

Baker is a library that provides a simple and intuitive way to orchestrate microservice-based process flows. You declare your orchestration logic as a **Recipe** using the Java, Kotlin, or Scala DSL. A recipe consists of **interactions** (system calls), **ingredients** (data), and **events**.

Baker's ability to visualize recipes provides a powerful communication tool that helps product owners, architects, and engineers have a common understanding of the business process. Baker allows for reuse of common interactions across different recipes, promoting consistency and reducing duplication.

### Current Version: 4.1.0

To use Baker you need three modules:
1. **recipe-dsl**: DSL to describe recipes in a declarative manner
2. **compiler**: Compiles recipes into models the runtime can execute
3. **runtime**: The runtime to manage and execute recipes

---

## 2. Core Concepts

### Ingredient

An ingredient is a combination of a **name** and **type**. Similar to how a variable declaration in your codebase is a combination of a name and type. For example, you can have an ingredient with the name `iban` and type `string`.

- Ingredients are **pure pieces of data** — immutable, meaning they do not change once they enter the process
- No support for hierarchy (expressing `Animal -> Dog -> Labrador` is not possible)
- Ingredients serve as **input for interactions** and are carried through the process via events

### Event

Events represent something that has happened in your process. Two types exist:

| Event Type | Description |
|------------|-------------|
| **Internal Event** | Output of an interaction |
| **Sensory Event** | Comes from outside the process; starts/triggers the process |

An event has a **name** and (optionally) provides **ingredients**. Example: an `OrderPlaced` event carrying `orderId` and `list of products`.

### Interaction

Interactions resemble functions. They:
- Require **input** (ingredients)
- Provide **output** (events)

An interaction can fetch data from another service, do complex calculations, send messages to an event broker, etc.

### Recipe

A recipe is the **blueprint of your business process**. You define this process by combining ingredients, events, and interactions using the Java, Kotlin, or Scala DSL.

### Specification vs Runtime Objects

| Specification | Runtime |
|---------------|---------|
| `Type` | `Value` |
| `Ingredient` | `IngredientInstance` |
| `Event` | `EventInstance` |
| `Interaction` | `InteractionInstance` |
| `Recipe` | `RecipeInstance` |

---

## 3. Configuration Reference

### Maven Dependencies

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `groupId` | String | `com.ing.baker` | Maven group ID for all Baker artifacts |
| `baker-recipe-dsl_2.13` | artifact | — | DSL to describe recipes declaratively |
| `baker-recipe-dsl-kotlin_2.13` | artifact | — | Kotlin DSL (use instead of regular DSL for Kotlin) |
| `baker-compiler_2.13` | artifact | — | Compiles recipes into executable models |
| `baker-runtime_2.13` | artifact | — | Runtime to manage and execute recipes |
| `baker-test_2.13` | artifact | — | Testing utilities (test scope) |

### Recipe Configuration Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `withEventReceivePeriod` | Duration | forever | Period during which the process accepts sensory events |
| `withRetentionPeriod` | Duration | forever | Period the process keeps running; deleted after this period (measured from creation) |
| `withDefaultFailureStrategy` | FailureStrategy | `BlockInteraction` | Default failure strategy for interactions without explicit strategy |

### Sensory Event Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `maxFiringLimit` | int | `1` | How many times the event can be fired into the process; use `unlimitedFiringLimit` for no limit |

### Interaction Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `withName` | String | class name | Custom name for the interaction |
| `withMaximumInteractionCount` | int | unlimited | Maximum number of times interaction can be invoked |
| `withPredefinedIngredients` | Map | — | Static ingredients registered to the interaction |
| `withRequiredEvents` | Event[] | — | Events that must be available (logical AND) for interaction to execute |
| `withRequiredOneOfEvents` | Event[] | — | At least one of these events must be available (logical OR) |
| `withFailureStrategy` | FailureStrategy | recipe default | Interaction-specific failure strategy (takes precedence) |

### Failure Strategy Configuration

| Strategy | Description |
|----------|-------------|
| `BlockInteraction` | Default. Blocks interaction on exception; must be unblocked via `baker.retryInteraction` or `baker.resolveInteraction` |
| `FireEventAndBlock(eventClass)` | Fires event on exception but still blocks interaction |
| `FireEventAndResolve(eventClass)` | Fires event on exception; interaction not blocked, can execute again |
| `RetryWithIncrementalBackoff` | Exponential backoff retry with configurable parameters |

### Retry With Incremental Backoff Parameters

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `initialDelay` | Duration | — | Delay before retrying for the first time |
| `backoffFactor` | double | `2` | Backoff factor for the delay |
| `maxTimeBetweenRetries` | Duration | — | Maximum time between retries |
| `deadline` | Duration | — | Total amount of time spent retrying (use `UntilDeadline`) |
| `maximumRetries` | int | — | Maximum amount of retries (use `UntilMaximumRetries`) |

### Akka Cluster Configuration

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `baker.actor.provider` | String | `local` | Set to `cluster-sharded` for cluster mode |
| `baker.cluster.nr-of-shards` | int | `52` | Number of shards for cluster |
| `akka.persistence.journal.plugin` | String | — | Use `cassandra-journal` for distributed store |
| `akka.persistence.snapshot-store.plugin` | String | — | Use `cassandra-snapshot-store` for distributed store |

---

## 4. Code Examples

### Example 1: Defining a Recipe (Java)

```java
final Recipe recipe = new Recipe("Web shop")
    .withSensoryEvents(
        CustomerInfoReceived.class,
        OrderPlaced.class,
        PaymentMade.class
    )
    .withInteractions(
        InteractionDescriptor.of(ValidateOrder.class),
        InteractionDescriptor.of(ReserveItems.class)
            .withRequiredEvent(PaymentMade.class),
        InteractionDescriptor.of(ShipGoods.class),
        InteractionDescriptor.of(SendInvoice.class)
            .withRequiredEvent(GoodsShipped.class)
    )
    .withDefaultFailureStrategy(
        new RetryWithIncrementalBackoffBuilder()
            .withInitialDelay(Duration.ofMillis(100))
            .withDeadline(Duration.ofHours(24))
            .withMaxTimeBetweenRetries(Duration.ofMinutes(10))
            .build());
```

### Example 2: Defining a Recipe (Kotlin)

```kotlin
val recipe = recipe("Web shop") {
    sensoryEvents {
        event<CustomerInfoReceived>()
        event<OrderPlaced>()
        event<PaymentMade>()
    }
    interaction<ValidateOrder>()
    interaction<ReserveItems> {
        requiredEvents {
            event<PaymentMade>()
        }
    }
    interaction<ShipGoods>()
    interaction<SendInvoice> {
        requiredEvents {
            event<GoodsShipped>()
        }
    }
    defaultFailureStrategy = retryWithIncrementalBackoff {
        initialDelay = 100.milliseconds
        until = deadline(24.hours)
        maxTimeBetweenRetries = 10.minutes
    }
}
```

### Example 3: Defining a Recipe (Scala)

```scala
val recipe: Recipe = Recipe("Web shop")
  .withSensoryEvents(
    Event[CustomerInfoReceived],
    Event[OrderPlaced],
    Event[PaymentMade]
  )
  .withInteractions(
    ValidateOrder,
    ReserveItems
      .withRequiredEvent(Event[PaymentMade]),
    ShipGoods,
    SendInvoice
      .withRequiredEvent(goodsShipped)
  )
  .withDefaultFailureStrategy(
    RetryWithIncrementalBackoff
      .builder()
      .withInitialDelay(100 milliseconds)
      .withUntil(Some(UntilDeadline(24 hours)))
      .withMaxTimeBetweenRetries(Some(10 minutes))
      .build()
  )
```

### Example 4: Defining a Sensory Event (Java)

```java
public class OrderPlaced {
    public final String orderId;
    public final String customerId;
    public final List<String> productIds;
    public final Address address;

    public OrderPlaced(String orderId, String customerId, 
                       List<String> productIds, Address address) {
        this.orderId = orderId;
        this.customerId = customerId;
        this.productIds = productIds;
        this.address = address;
    }
}
```

### Example 5: Defining an Interaction Interface (Java)

```java
public interface CheckStock extends Interaction {

    interface CheckStockOutcome {}

    class SufficientStock implements CheckStockOutcome {}

    class OrderHasUnavailableItems implements CheckStockOutcome {
        public final List<String> unavailableProductIds;
        
        public OrderHasUnavailableItems(List<String> unavailableProductIds) {
            this.unavailableProductIds = unavailableProductIds;
        }
    }

    @FiresEvent(oneOf = {SufficientStock.class, OrderHasUnavailableItems.class})
    CheckStockOutcome apply(
        @RequiresIngredient("orderId") String orderId,
        @RequiresIngredient("productIds") List<String> productIds
    );
}
```

### Example 6: Implementing an Interaction (Java)

```java
public class CheckStockImpl implements CheckStock {
    @Override
    public CheckStockOutcome apply(String orderId, List<String> productIds) {
        // Call stock service
        boolean inStock = stockService.checkAvailability(productIds);
        
        if (inStock) {
            return new SufficientStock();
        } else {
            return new OrderHasUnavailableItems(productIds);
        }
    }
}
```

### Example 7: Creating and Running Baker (Java)

```java
// Create interaction instances
InteractionInstance checkStockInstance = InteractionInstance.from(new CheckStockImpl());
InteractionInstance shipOrderInstance = InteractionInstance.from(new ShipOrderImpl());
InteractionInstance cancelOrderInstance = InteractionInstance.from(new CancelOrderImpl());

// Create in-memory Baker
Baker baker = InMemoryBaker.java(
    List.of(checkStockInstance, shipOrderInstance, cancelOrderInstance)
);

// Add recipe
RecipeRecord recipeRecord = RecipeRecord.of(
    RecipeCompiler.compileRecipe(webShopRecipe), 
    true  // validate interactions
);
String recipeId = baker.addRecipe(recipeRecord).join();

// Create recipe instance (bake)
String recipeInstanceId = UUID.randomUUID().toString();
baker.bake(recipeId, recipeInstanceId).join();

// Fire sensory event
EventInstance orderPlaced = EventInstance.from(
    new OrderPlaced("order-123", "customer-456", 
                    List.of("iPhone", "PlayStation5"), 
                    new Address("Hoofdstraat", "Amsterdam", "1234AA", "Netherlands"))
);
baker.fireSensoryEventAndAwaitReceived(recipeInstanceId, orderPlaced).join();

// Wait for completion
baker.awaitCompleted(recipeInstanceId).join();
```

### Example 8: Retry With Backoff and Fire Event (Java)

```java
Recipe recipe = new Recipe("OrderProcess")
    .withSensoryEvents(OrderPlaced.class)
    .withInteractions(
        InteractionDescriptor.of(ProcessPayment.class)
    )
    .withDefaultFailureStrategy(
        new RetryWithIncrementalBackoffBuilder()
            .withInitialDelay(Duration.ofMillis(100))
            .withMaxTimeBetweenRetries(Duration.ofSeconds(100))
            .withDeadline(Duration.ofHours(24))
            .withFireRetryExhaustedEvent(PaymentFailed.class)
            .build()
    );
```

### Example 9: Checkpoint Events (Java)

```java
Recipe recipe = new Recipe("OrderProcess")
    .withSensoryEvents(OrderPlaced.class)
    .withInteractions(
        InteractionDescriptor.of(ValidateOrder.class),
        InteractionDescriptor.of(ProcessPayment.class)
    )
    .withCheckpointEvent(
        CheckPointEvent.builder()
            .withName("OrderValidatedAndPaid")
            .withRequiredEvents(OrderValidated.class, PaymentReceived.class)
            .build()
    );
```

### Example 10: Sub-Recipes (Java)

```java
// Define a sub-recipe (building block)
Recipe paymentSubRecipe = new Recipe("PaymentSubRecipe")
    .withInteractions(
        InteractionDescriptor.of(ValidatePayment.class),
        InteractionDescriptor.of(ProcessPayment.class)
    );

// Main recipe including sub-recipe
Recipe mainRecipe = new Recipe("MainOrderRecipe")
    .withSensoryEvents(OrderPlaced.class)
    .withInteractions(
        InteractionDescriptor.of(ValidateOrder.class)
    )
    .withSubRecipe(paymentSubRecipe);
```

### Example 11: Testing with baker-test (Java)

```java
// Define expected events flow
EventsFlow happyFlow = EventsFlow.of(
    OrderPlaced.class,
    SufficientStock.class,
    OrderShipped.class
);

// Assert events and ingredients
RecipeAssert.of(baker, recipeInstanceId)
    .waitFor(happyFlow)
    .assertEventsFlow(happyFlow)
    .assertIngredient("orderId").isEqual("order-123")
    .assertIngredient("shippingStatus").isEqual("SHIPPED");
```

### Example 12: Visualizing a Recipe (Java)

```java
CompiledRecipe compiledRecipe = RecipeCompiler.compileRecipe(webShopRecipe);

// Get Graphviz visualization string
String graphvizString = compiledRecipe.getRecipeVisualization();

// For sub-recipes, get high-level view
String subRecipeView = compiledRecipe.getSubRecipeVisualization();

// Get visual state of running instance
String visualState = baker.getVisualState(recipeInstanceId).join();
```

### Example 13: Manual Intervention (Java)

```java
// Force retry a blocked interaction
baker.retryInteraction(recipeInstanceId, "ProcessPayment").join();

// Resolve blocked interaction by firing an event
EventInstance resolveEvent = EventInstance.from(new PaymentManuallyApproved());
baker.resolveInteraction(recipeInstanceId, "ProcessPayment", resolveEvent).join();

// Stop retrying an interaction (blocks it)
baker.stopRetryingInteraction(recipeInstanceId, "ProcessPayment").join();
```

### Example 14: Event Listeners (Java)

```java
// Register listener for specific recipe instance events
baker.registerEventListener(recipeInstanceId, (metadata, event) -> {
    System.out.println("Event fired: " + event.getName());
});

// Register listener for all Baker events
baker.registerBakerEventListener((bakerEvent) -> {
    if (bakerEvent instanceof InteractionStarted) {
        System.out.println("Interaction started: " + 
            ((InteractionStarted) bakerEvent).getInteractionName());
    }
});
```

### Example 15: Async Interactions (Scala)

```scala
trait ReserveItems {
  def apply(orderId: String, items: List[String]): Future[ReserveItemsOutput]
}

class ReserveItemsInstance extends ReserveItems {
  override def apply(orderId: String, items: List[String]): Future[ReserveItemsOutput] = {
    // Http call to the Warehouse service
    val response: Future[Either[List[String], List[String]]] =
      warehouseService.reserve(items)

    response.map {
      case Left(unavailableItems) => OrderHadUnavailableItems(unavailableItems)
      case Right(reservedItems) => ItemsReserved(reservedItems)
    }
  }
}

val reserveItemsInstance: InteractionInstance =
  InteractionInstance.unsafeFrom(new ReserveItemsInstance)
```

---

## 5. Integration with Other ING Tools

### Akka Integration

Baker is built on top of Akka and leverages:
- **Akka Persistence** for event sourcing and state recovery
- **Akka Cluster Sharding** for distributed recipe instances
- **Akka Serialization** for distributed communication

### Cassandra Integration

For production cluster deployments, Baker requires a distributed data store. Cassandra is the recommended choice:

```hocon
akka.persistence {
  journal.plugin = "cassandra-journal"
  snapshot-store.plugin = "cassandra-snapshot-store"
}
```

### Kubernetes Deployment

Baker cluster mode provides:
- **Elasticity**: Add/remove nodes dynamically
- **Resilience**: RecipeInstances automatically restored on node failure
- **Routing**: Fire events from anywhere in the cluster

⚠️ For production Kubernetes deployments, configure appropriate:
- Cluster seed nodes
- Cassandra connection settings
- Split-brain resolver

---

## 6. Pitfalls & Anti-patterns

❌ **Don't**: Use firing limit of 1 for events that may need to be re-sent
✅ **Do**: Explicitly set `unlimitedFiringLimit` if events can fire multiple times

❌ **Don't**: Block interactions for idempotent operations that can be safely retried
✅ **Do**: Use `RetryWithIncrementalBackoff` for transient failures

❌ **Don't**: Use non-idempotent interactions with retry strategies
✅ **Do**: Use `BlockInteraction` strategy for non-idempotent interactions that cannot be safely retried

❌ **Don't**: Forget that interaction execution time is NOT included in retry deadlines
✅ **Do**: Account for interaction execution time when setting deadline values

❌ **Don't**: Use local mode with in-memory journal in production
✅ **Do**: Configure Cassandra persistence for production cluster deployments

❌ **Don't**: Rely on event listeners for critical functionality
✅ **Do**: Understand event listeners deliver AT MOST ONCE; use for monitoring/logging only

❌ **Don't**: Assume event listeners receive events from other cluster nodes
✅ **Do**: Remember event listener delivery is local (JVM) only

❌ **Don't**: Name interaction methods anything other than `apply` when using reflection API
✅ **Do**: Always name your interaction method `apply` for Java/Kotlin reflection to work

❌ **Don't**: Create RecipeInstances with duplicate IDs
✅ **Do**: Use UUIDs or other unique identifiers for recipeInstanceId

❌ **Don't**: Expect sub-recipes to inherit configuration (retention periods, failure strategies)
✅ **Do**: Define configuration settings on the main top-level recipe only; sub-recipes only contribute interactions

---

## 7. FAQ

**Q: What's the difference between internal events and sensory events?**
A: They are technically identical. The naming distinction is practical: sensory events come from outside the process and typically trigger it, while internal events are outputs of interactions within the recipe.

**Q: Can an interaction be executed multiple times?**
A: Yes, by default interactions can execute unlimited times. Use `withMaximumInteractionCount` to limit this. An interaction re-executes when its preconditions (required ingredients and events) are met again.

**Q: How do I handle functional failures vs technical failures?**
A: Technical failures (timeouts, unavailable services) should throw exceptions and use retry strategies. Functional failures (insufficient stock, invalid input) should return specific events that the recipe handles as normal flow.

**Q: When should I use `FireEventAndBlock` vs `FireEventAndResolve`?**
A: Use `FireEventAndBlock` when you want to fire an event on failure but still require manual intervention. Use `FireEventAndResolve` when the event itself resolves the situation and the interaction should be available to execute again if preconditions are met.

**Q: How do I test recipes without real external services?**
A: Create mock/dummy interaction implementations that return expected events. Use the `baker-test` library's `RecipeAssert` and `EventsFlow` for clean assertions.

**Q: What happens when a node fails in cluster mode?**
A: With properly configured Cassandra persistence, RecipeInstances are automatically restored on a new node. Events can be fired from any node, and Baker routes them correctly.

**Q: Can I use Kotlin coroutines with Baker?**
A: Baker supports suspending Kotlin APIs for the runtime. However, note that `baker-test` is currently not compatible with suspending Kotlin Baker APIs.

---

## 8. Glossary

| Term | Definition |
|------|------------|
| **Recipe** | The blueprint of a business process, combining ingredients, events, and interactions |
| **Ingredient** | Immutable data (name + type) that flows through the process |
| **Event** | A happening in the process that may carry ingredients; either sensory (external) or internal (from interactions) |
| **Interaction** | A function that takes ingredients as input and produces events as output |
| **Sensory Event** | An event that comes from outside the recipe, typically used to start or trigger the process |
| **Internal Event** | An event produced by an interaction within the recipe |
| **RecipeInstance** | A running instance of a Recipe, created via `baker.bake()` |
| **InteractionInstance** | A runtime implementation of an Interaction interface |
| **Firing Limit** | The maximum number of times a sensory event can be fired into a process |
| **Checkpoint Event** | An event automatically fired when specified preconditions (events) are met |
| **Blocked Interaction** | An interaction that failed and is waiting for manual intervention |
| **CompiledRecipe** | A Recipe after compilation, ready to be added to Baker runtime |
| **Baker Runtime** | The engine that manages and executes recipes (InMemoryBaker or AkkaBaker) |
| **Event Sourcing** | The persistence mechanism Baker uses to store and replay RecipeInstance state |
| **Sub-Recipe** | A recipe used as a building block within another recipe; only contributes interactions |
