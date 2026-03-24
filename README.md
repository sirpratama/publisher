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

    In this BambangShop case, a single `Subscriber` model struct is sufficient. We do not need a trait (interface) because all subscribers behave the same way — they all have a `url` and a `name`, and they all receive notifications through the same HTTP POST mechanism. A trait would be useful if we had multiple types of subscribers with different notification behaviors (e.g., email subscriber, SMS subscriber, webhook subscriber), but since all subscribers in BambangShop are notified via the same HTTP endpoint pattern, a single struct adequately represents the observer.

2. **id** in `Program` and **url** in `Subscriber` are intended to be unique. Explain based on your understanding, is using **Vec** (list) sufficient or using **DashMap** (map/dictionary) necessary for this case?

    Using a `DashMap` is necessary rather than a `Vec`. Since `id` and `url` must be unique, we need a data structure that enforces or naturally supports uniqueness. With a `Vec`, we would need to iterate through the entire list every time we want to check for duplicates, find, or delete a subscriber — resulting in O(n) time complexity for these operations. A `DashMap` provides O(1) average-case lookups, insertions, and deletions by key, making it more efficient and naturally suitable for enforcing uniqueness.

3. When programming using Rust, we are enforced by rigorous compiler constraints to make a thread-safe program. In the case of the List of Subscribers (**SUBSCRIBERS**) static variable, we used the **DashMap** external library. Explain based on your understanding of design patterns, do we still need **DashMap** or we can implement Singleton pattern instead?

    We still need `DashMap` even with a Singleton pattern. The Singleton pattern ensures there is only one instance of the subscribers collection, but it does not by itself guarantee thread-safe concurrent access. `DashMap` provides built-in thread-safe concurrent read and write operations without requiring explicit locking. If we used a regular `HashMap` wrapped in a `Mutex` or `RwLock`, it would work but with worse performance under concurrent access because the entire map would be locked. `DashMap` uses sharded locking internally, allowing multiple threads to read/write different parts of the map simultaneously. In Rust's `lazy_static!`, we already achieve the Singleton pattern — `DashMap` complements it by providing safe concurrent access.

#### Reflection Publisher-2

1. In the Model-View Controller (MVC) compound pattern, there is no "Service" and "Repository". Model in MVC covers both data storage and business logic. In this module, we separate "Service" and "Repository" from a Model. Explain based on your understanding of design principles, why do we need to separate "Service" and "Repository" from a Model?

    Separating Service and Repository from the Model follows the **Single Responsibility Principle (SRP)** and **Separation of Concerns**. The Model should only represent the data structure. The Repository handles data access and persistence logic (CRUD operations on the data store), while the Service handles business logic and orchestration. Without this separation, the Model would become bloated with responsibilities — it would need to know how to store itself, validate business rules, and represent data all at once. By separating them, each layer has a single reason to change: the Model changes when the data structure changes, the Repository changes when the storage mechanism changes, and the Service changes when business rules change.

2. What happens if we only use the Model without separating "Service" and "Repository"? Explain your imagination on how the interactions between each model (`Program`, `Subscriber`, `Notification`) affect the code complexity for each model?

    If we only use the Model, each model would contain data access logic, business logic, and inter-model interactions all in one place. For example, `Product` would need to know how to store itself, fetch subscribers, create notifications, and send HTTP requests. `Subscriber` would need to know how to register itself in storage and handle incoming notifications. `Notification` would need to know about both products and subscribers. This creates tight coupling between models — changing one model's logic would ripple through the others. The code complexity grows quadratically because every model depends on every other model directly, making it extremely hard to maintain, test, and extend.

3. Have you explored things outside of the steps in the tutorial, for example: `src/lib.rs`? If not, explain why you did not do so; if yes, explain things that you have learned from those other parts of the code.

    Yes, I explored `src/lib.rs`. It defines the application-level configuration and utilities. The `AppConfig` struct uses `getset` for auto-generating getters and `dotenvy` for loading environment variables from `.env` files. The `lazy_static!` macro is used to create global singletons for the `REQWEST_CLIENT` (HTTP client for making requests to subscribers) and `APP_CONFIG`. The file also defines a custom `Result` type alias and `Error` type using Rocket's `Custom` response wrapper, along with a `compose_error_response` helper function. This pattern centralizes error handling and configuration, making it reusable across all services and controllers.

#### Reflection Publisher-3

1. Observer Pattern has two variations: **Push model** (publisher pushes data to subscribers) and **Pull model** (subscribers pull data from publisher). In this tutorial case, which variation of Observer Pattern that we use?

    This tutorial uses the **Push model**. When a product is created, deleted, or promoted, the publisher (main app) actively pushes notification data to each subscriber by sending HTTP POST requests to each subscriber's URL. The subscribers do not need to request or poll for updates — the publisher takes the initiative to push the `Notification` payload containing all relevant information (product title, type, URL, subscriber name, status) directly to each subscriber's `/receive` endpoint.

2. What are the advantages and disadvantages of using the other variation of Observer Pattern for this tutorial case? (example: if you answer Push, then what are the advantages and disadvantages of Pull)

    If we used the **Pull model** instead:

    **Advantages:**
    - Subscribers have more control over when they retrieve data, reducing unnecessary processing when they're busy.
    - The publisher is simpler since it only needs to notify subscribers that something changed, without sending full payload data.
    - Subscribers can selectively pull only the data they need, reducing bandwidth usage.

    **Disadvantages:**
    - Subscribers need to make additional HTTP requests to the publisher to fetch the actual data after being notified, increasing latency and network overhead.
    - The publisher needs to expose additional endpoints for subscribers to pull data from.
    - Subscribers may miss time-sensitive notifications if they don't pull frequently enough.
    - More complex implementation on the subscriber side — each subscriber needs to know how to query the publisher for relevant data.

3. Explain what will happen to the program if we decide to not use multi-threading in the notification process.

    If we do not use multi-threading (`thread::spawn`) in the notification process, the `notify` function would send HTTP POST requests to each subscriber **sequentially** on the main thread. This means each notification must complete (or timeout) before the next one is sent. If there are many subscribers or if a subscriber's server is slow to respond, the entire `notify` process would block the main application thread. This would cause the API response (e.g., product creation or deletion) to take a very long time to return, because the server has to wait for all subscriber notifications to finish before responding to the original request. In the worst case, a single unresponsive subscriber could delay the entire system. With multi-threading, each notification is sent in a separate thread, allowing them to execute concurrently and the main thread to return a response immediately.
