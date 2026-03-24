# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

1. In the Observer pattern diagram explained in the Head First Design Patterns book, Subscriber is defined as an interface. Explain based on your understanding of Observer design patterns, do we still need an interface (or **trait** in Rust) in this BambangShop case, or a single Model **struct** is enough?

    In this BambangShop case, a single `Subscriber` model struct is enough. We don't really need a trait (interface) because all subscribers behave the same way. They all have a `url` and a `name`, and they all receive notifications through the same HTTP POST mechanism. A trait would make more sense if we had different types of subscribers with different notification behaviors, like an email subscriber vs an SMS subscriber vs a webhook subscriber. But since all subscribers in BambangShop are notified via the same HTTP endpoint pattern, using just one struct is sufficient to represent the observer.

2. **id** in `Program` and **url** in `Subscriber` are intended to be unique. Explain based on your understanding, is using **Vec** (list) sufficient or using **DashMap** (map/dictionary) necessary for this case?

    Using a `DashMap` is necessary here rather than a `Vec`. Since `id` and `url` are meant to be unique, we need a data structure that naturally supports uniqueness. If we used a `Vec`, we'd have to loop through the entire list every time we want to check for duplicates, find a specific subscriber, or delete one, which gives us O(n) time complexity. A `DashMap` on the other hand provides O(1) average-case lookups, insertions, and deletions by key, so it's way more efficient and also naturally prevents duplicate keys.

3. When programming using Rust, we are enforced by rigorous compiler constraints to make a thread-safe program. In the case of the List of Subscribers (**SUBSCRIBERS**) static variable, we used the **DashMap** external library. Explain based on your understanding of design patterns, do we still need **DashMap** or we can implement Singleton pattern instead?

    We still need `DashMap` even if we use a Singleton pattern. The Singleton pattern only makes sure there's one instance of the subscribers collection, but it doesn't guarantee thread-safe concurrent access by itself. `DashMap` gives us built-in thread-safe concurrent read and write operations without needing explicit locking. If we were to use a regular `HashMap` wrapped in a `Mutex` or `RwLock`, it would technically work, but performance would be worse under concurrent access since the whole map gets locked at once. `DashMap` uses sharded locking internally, so multiple threads can read/write to different parts of the map at the same time. In our code, `lazy_static!` already gives us the Singleton pattern, and `DashMap` complements that by handling safe concurrent access.

#### Reflection Publisher-2

1. In the Model-View Controller (MVC) compound pattern, there is no "Service" and "Repository". Model in MVC covers both data storage and business logic. In this module, we separate "Service" and "Repository" from a Model. Explain based on your understanding of design principles, why do we need to separate "Service" and "Repository" from a Model?

    Separating Service and Repository from the Model follows the Single Responsibility Principle (SRP) and Separation of Concerns. The Model should only represent the data structure itself. The Repository is responsible for data access and persistence logic (like CRUD operations on the data store), while the Service handles business logic and orchestration. Without this separation, the Model would get bloated with too many responsibilities since it would need to handle storing itself, validating business rules, and representing data all at the same time. By separating them, each layer only has one reason to change: the Model changes when the data structure changes, the Repository changes when the storage mechanism changes, and the Service changes when business rules change.

2. What happens if we only use the Model without separating "Service" and "Repository"? Explain your imagination on how the interactions between each model (`Program`, `Subscriber`, `Notification`) affect the code complexity for each model?

    If we only use the Model, each model would end up containing data access logic, business logic, and inter-model interactions all in one place. For example, `Product` would need to handle storing itself, fetching subscribers, creating notifications, and sending HTTP requests. `Subscriber` would need to handle registering itself in storage and dealing with incoming notifications. `Notification` would need to know about both products and subscribers. This creates tight coupling between models, so changing one model's logic would have a ripple effect on the others. The code complexity would grow a lot because every model depends on every other model directly, making it really hard to maintain, test, and extend.

3. Have you explored things outside of the steps in the tutorial, for example: `src/lib.rs`? If not, explain why you did not do so; if yes, explain things that you have learned from those other parts of the code.

    Yes, I took a look at `src/lib.rs`. It basically defines the application-level configuration and shared utilities. The `AppConfig` struct uses `getset` to auto-generate getters and `dotenvy` to load environment variables from `.env` files. There's also `lazy_static!` which creates global singletons for `REQWEST_CLIENT` (the HTTP client used to send requests to subscribers) and `APP_CONFIG`. On top of that, the file defines a custom `Result` type alias and `Error` type using Rocket's `Custom` response wrapper, plus a `compose_error_response` helper function. Having all of this centralized in `lib.rs` makes it reusable across all the services and controllers.

#### Reflection Publisher-3

1. Observer Pattern has two variations: **Push model** (publisher pushes data to subscribers) and **Pull model** (subscribers pull data from publisher). In this tutorial case, which variation of Observer Pattern that we use?

    This tutorial uses the Push model. When a product is created, deleted, or promoted, the publisher (main app) actively pushes notification data to each subscriber by sending HTTP POST requests to their URLs. The subscribers don't need to request or poll for updates at all. The publisher takes the initiative to push the `Notification` payload containing all the relevant information (product title, type, URL, subscriber name, status) directly to each subscriber's `/receive` endpoint.

2. What are the advantages and disadvantages of using the other variation of Observer Pattern for this tutorial case? (example: if you answer Push, then what are the advantages and disadvantages of Pull)

    If we used the Pull model instead:

    Advantages:
    - Subscribers have more control over when they retrieve data, so they can avoid unnecessary processing when they're busy.
    - The publisher becomes simpler since it only needs to notify subscribers that something changed, without sending the full payload data.
    - Subscribers can selectively pull only the data they actually need, which can reduce bandwidth usage.

    Disadvantages:
    - Subscribers need to make additional HTTP requests to the publisher to fetch the actual data after being notified, which increases latency and network overhead.
    - The publisher would need to expose additional endpoints for subscribers to pull data from.
    - Subscribers might miss time-sensitive notifications if they don't pull frequently enough.
    - The implementation becomes more complex on the subscriber side since each subscriber needs to know how to query the publisher for the relevant data.

3. Explain what will happen to the program if we decide to not use multi-threading in the notification process.

    Without multi-threading (`thread::spawn`), the `notify` function would send HTTP POST requests to each subscriber one by one on the main thread. So each notification has to finish (or timeout) before the next one gets sent. If there are a lot of subscribers or if one subscriber's server is slow to respond, the whole `notify` process blocks the main application thread. This means the API response for things like product creation or deletion would take a really long time to return, since the server is stuck waiting for all notifications to complete before it can respond to the original request. Worst case, a single unresponsive subscriber could hold up everything. With multi-threading, each notification gets its own thread so they all run concurrently, and the main thread can return a response right away.
