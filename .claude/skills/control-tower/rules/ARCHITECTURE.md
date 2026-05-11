# Clean Architecture Best Practices

## History

- **Origin:** First introduced by Robert C. Martin (Uncle Bob) on his blog in 2012. It is characterized by integrating the strengths of existing architectures like Hexagonal and Onion Architecture, clearly defining them into a four-concentric-circle structure.

---

## Applicable Scope

Clean Architecture is a design paradigm that strictly separates software layers, ensuring that business rules do not depend on the UI, database, web frameworks, etc.

- **Server (Backend):** Serves as the de facto standard skeleton when building enterprise-level systems in Spring Boot, Node.js (NestJS), Go, and more.
- **Client (Mobile, Web & Desktop):** Highly active in the Android (MVI+Clean) and iOS (VIPER, etc.) ecosystems. Even when developing native desktop apps with Swift in your **macOS** environment or creating web frontend (React) apps, it is excellently applied to maximize reusability by separating state management (business logic) and UI (View).

---

## Practical Application

While Clean Architecture seems perfect in theory, blindly dividing your application into 4 layers can severely degrade development speed. Here are best practices to apply in real-world scenarios.

### Scenario: "Overcoming Productivity Decline Due to Excessive Object Mapping"

In a strict Clean Architecture, when a user requests data, numerous object conversions are forced: `DB Entity -> Domain Entity -> UseCase Response -> Controller DTO -> ViewModel`.

- **Best Practice:** "Partial Adoption of the CQRS (Command and Query Responsibility Segregation) Pattern"
  - **When modifying data (Command - Create, Update, Delete):** Data integrity is crucial, so it strictly passes through the 4 layers of Clean Architecture and undergoes validation by the business logic (Entities, Use Cases).
  - **When simply querying data (Query - Read):** If it's a simple list retrieval without complex business logic, bypass the Use Case or Domain layers and directly query the database (or a read-only view) from the Interface Adapter layer, returning a DTO.
  - **Effect:** This thoroughly protects core business logic while reducing unnecessary boilerplate code, thereby increasing development productivity.

---

## The Dependency Rule

This is the single absolute rule that permeates Clean Architecture.

> "Source code dependencies must point only inward, toward higher-level policies (business logic)."

This means that code belonging to the inner layers (Core) must not know anything about the outer layers. Classes, functions, variables, and even data formats declared in the outer layers must not be referenced (imported) by the inner layers.

### Why?

Let's assume you are developing an app on macOS. Initially, you built it as a CLI (Command Line Interface) environment running in the terminal, but later you wanted to change it to an elegant SwiftUI-based desktop GUI app. If you strictly followed the dependency rule, you could completely overhaul the UI (outer layer) while reusing the core business logic (inner layer) as is, without modifying a single line of code.

---

## Roles and Responsibilities of the 4 Layers

Clean Architecture divides the system into four concentric layers. The further in you go, the closer you get to the essence of the software (high-level), and the further out you go, the closer you get to specific technologies (low-level).

External clients (web browsers, mobile UIs, etc.) must access the system solely through the `Frameworks & Drivers` layer. A proper design strictly blocks external entities from directly referencing or calling the other folders (`Use Cases`, `Entities`, `Interface Adapters`).

### Entities (Enterprise Business Rules)

- **Role:** Contains the most core domain objects and business rules of the software. These are essential rules that would remain unchanged even if the tasks were handled with just paper and pencil, without a computer system.
- **Characteristics:** They know nothing about any outer layers (DB, UI, frameworks). They are the most stable and have the lowest probability of change.
- **Example:** An `Account` object in a banking system, 'loan interest calculation formulas', etc.

### Use Cases (Application Business Rules)

- **Role:** Defines the specific actions (scenarios) the system performs for the user. It manipulates entities to achieve a specific business goal.
- **Characteristics:** Unaffected by changes in outer layers (UI or DB), but if the system's functional requirements change (e.g., "From now on, add email verification upon signup"), this layer is modified.
- **Example:** `TransferMoneyUseCase`, `RegisterUserUseCase`, etc.

### Interface Adapters

- **Role:** Acts as a **converter** between data formats most convenient for the inner layers (Entities, Use Cases) and formats most convenient for the outer layers (DB, Web).
- **Characteristics:** The commonly known Controllers and Presenters of the MVC pattern, as well as Gateways (Repository implementations) that map DB data, belong here.
- **Example:** A `UserController` that takes an HTTP request and converts it into arguments suitable for a Use Case; a `UserPresenter` that takes the results of a Use Case and converts them into JSON.

### Frameworks & Drivers

- **Role:** Located on the outermost edge, these are the detailed technologies that connect the system to the outside world.
- **Characteristics:** Core business logic should not be placed in this layer; it primarily serves as 'glue code' to communicate with the inner circles.
- **Example:** SwiftUI (UI framework) running on your **macOS**, Spring Boot (web framework), MySQL (database), AWS S3 (external storage), etc.

---

## Understanding Control Flow and Dependency Inversion Principle (DIP)

From the moment a client requests data until it is saved in the DB, the program's control flow is as follows:

> `Presentation (Controller)` ➡️ `Application (Use Case)` ➡️ `Infrastructure (DB)`

However, if the direction of code references (imports)—the dependency flow—strictly follows the control flow, a fatal problem occurs where the inner layers depend on the outer DB layer.

To solve this, the Dependency Inversion Principle (DIP) is used: placing an interface (Port) in the inner layer and having the outer layer implement it (Adapter), thereby bending the dependency direction inward.

### Rust Code Example (Dependency Inversion and Injection)

In Rust, you define a Port (interface) using a `trait`, and create an Adapter (implementation) using a `struct`.

#### Step 1: Inner Layer - Domain and Application (Use Case & Port)

This code is pure business logic. There should be no `use` statements for the outside world (DB, Web).

Rust

```rust
// domain/user.rs
pub struct User {
    pub id: String,
    pub name: String,
}

// application/port/out.rs (Output Port)
// This is the 'contract' that Application demands from Infrastructure.
pub trait UserRepository: Send + Sync {
    fn save(&self, user: &User) -> Result<(), String>;
}

// application/service/register_user.rs (Use Case)
// Business logic depends solely on the Port (trait).
use crate::domain::user::User;
use crate::application::port::out::UserRepository;
use std::sync::Arc;

pub struct RegisterUserUseCase {
    // We use dynamic dispatch (Arc<dyn Trait>) for dependency injection.
    user_repository: Arc<dyn UserRepository>,
}

impl RegisterUserUseCase {
    pub fn new(user_repository: Arc<dyn UserRepository>) -> Self {
        Self { user_repository }
    }

    pub fn execute(&self, id: String, name: String) -> Result<(), String> {
        let new_user = User { id, name };
        // We don't know if the DB is PostgreSQL or in-memory, but we issue the save command according to the contract (Port).
        self.user_repository.save(&new_user)
    }
}
```

#### Step 2: Outer Layer - Infrastructure (Adapter)

The outer layer brings in (depends on) the `UserRepository` trait defined inside and implements it with actual technology.

Rust

```rust
// infrastructure/persistence/postgres_adapter.rs
use crate::domain::user::User;
use crate::application::port::out::UserRepository;

pub struct PostgresUserRepository {
    // Space where the actual DB connection pool would go
    // pool: sqlx::PgPool,
}

impl PostgresUserRepository {
    pub fn new() -> Self {
        Self {}
    }
}

// The outer layer implements the inner layer's Port (Dependency Inversion occurs!)
impl UserRepository for PostgresUserRepository {
    fn save(&self, user: &User) -> Result<(), String> {
        println!("Successfully saved user [{}] to local PostgreSQL DB on macOS!", user.name);
        // Actual SQL query execution logic...
        Ok(())
    }
}
```

#### Step 3: Assembly and Execution (Dependency Injection)

All objects are created and assembled at the application's entry point (`main.rs` where `cargo run` is executed in the macOS terminal).

Rust

```rust
// main.rs
mod domain;
mod application;
mod infrastructure;

use std::sync::Arc;
use application::service::register_user::RegisterUserUseCase;
use infrastructure::persistence::postgres_adapter::PostgresUserRepository;

fn main() {
    // 1. Create the outer layer's adapter (implementation)
    let postgres_repo = PostgresUserRepository::new();

    // 2. Inject the adapter into the inner layer's Use Case (Dependency Injection)
    // Allocate on the heap via Arc and pass the pointer.
    let use_case = RegisterUserUseCase::new(Arc::new(postgres_repo));

    // 3. Execute (Assuming the Controller in the Presentation layer calls this)
    match use_case.execute("user-123".to_string(), "Gemini".to_string()) {
        Ok(_) => println!("Signup successful!"),
        Err(e) => println!("Error occurred: {}", e),
    }
}
```

---

### DI Optimization Scenario in a Rust Environment

When implementing Clean Architecture in Rust, developers often agonize over: "Should I use Generics (Static Dispatch) or `dyn Trait` (Dynamic Dispatch) for Dependency Injection (DI)?"

- **Scenario:** Complex business logic requiring the injection of multiple Repositories and external API Clients into a single Use Case.
- **Bad Practice:** Unconditionally defining structs using Generics (`<T: UserRepository>`). The type signatures become exponentially complex, and compile times on macOS become excessively long (Monomorphization).
- **Best Practice Suggestion:** "Use `Arc<dyn Trait>` (Dynamic Dispatch) as the default for Port injection."
  - **Reason:** Clean Architecture's dependency injection is not frequently swapped at runtime; it is assembled once at application startup (`main.rs`) and that's it. The slight performance overhead from dynamic dispatch (vtable lookups) is completely negligible compared to network or DB I/O times in an enterprise environment.
  - **Effect:** Code readability improves tremendously, compile speeds are faster, and writing unit tests utilizing `Mock`objects becomes much easier.

---

## Inter-Layer Data Mapping (DTO Conversion Strategy)

In Clean Architecture, each layer must have its own data structure. The object mapped 1:1 with a DB table, the core business object of the domain, and the object responding as JSON to web clients must all be different.

### Bad Practice: Sharing Entities

Allowing database framework macros (`sqlx`, `diesel`, etc.) to invade domain entities.

Rust

```rust
// domain/user.rs
#[derive(serde::Serialize, sqlx::FromRow)] // Domain is polluted by outer technology (Web, DB)!
pub struct User {
    pub id: String,
    pub name: String,
}
```

### Best Practice: Isolation Using From / Into Traits

Separate models by layer and implement Rust's standard conversion traits for very clean conversions.

Rust

```rust
// 1. Domain Layer (Pure Business Object)
// domain/user.rs
pub struct User {
    pub id: String,
    pub name: String,
}

// 2. Infrastructure Layer (DB-Exclusive Object)
// infrastructure/persistence/user_db_model.rs
use crate::domain::user::User;

#[derive(sqlx::FromRow)] // DB-exclusive macros exist only here
pub struct UserDbModel {
    pub db_id: String,
    pub full_name: String,
    pub created_at: chrono::DateTime<chrono::Utc>, // Metadata existing only in the DB
}

// Defining the conversion rule from DB Model -> Domain Model
impl From<UserDbModel> for User {
    fn from(model: UserDbModel) -> Self {
        Self {
            id: model.db_id,
            name: model.full_name,
        }
    }
}

// 3. Presentation Layer (Web-Exclusive Object)
// presentation/dto/user_response.rs
use crate::domain::user::User;

#[derive(serde::Serialize)] // JSON serialization macros exist only here
pub struct UserResponseDto {
    pub user_id: String,
    pub display_name: String,
}

// Defining the conversion rule from Domain Model -> Web DTO
impl From<User> for UserResponseDto {
    fn from(user: User) -> Self {
        Self {
            user_id: user.id,
            display_name: user.name,
        }
    }
}
```

By implementing `From` like this, you can elegantly convert types in actual Use Cases or Controllers with just a single `.into()` method.

Rust

```rust
// Example of Web Controller logic in the Presentation Layer
let domain_user = get_user_usecase.execute("123");
// Magically converts the domain object into a web DTO!
let response_dto: UserResponseDto = domain_user.into();
```

---

## Error Handling Boundaries (Error Handling Strategy)

Not only data but also errors must cross boundaries. If an `sqlx::Error::RowNotFound` error occurs in the DB and is thrown as-is to the Presentation layer (Web API), the user will see an incomprehensible DB error message, creating a security vulnerability where the system's internal structure is exposed externally.

#### Best Practice: "Error Translation Scenario"

Each layer must define its own error types, and the outer layer must **translate** the inner layer's errors into errors appropriate for the caller.

1. **Defining Domain Layer Errors:** Define errors for business rule violations.

   Rust

   ```rust
   // domain/error.rs
   pub enum DomainError {
       UserNotFound,
       InvalidUserName,
       // Doesn't know the specific infrastructure error, but signals that something went wrong
       InternalSystemError,
   }
   ```

2. **Infrastructure Layer Error Translation (using `map_err`):** Translate DB errors into Domain errors and return them to the Use Case.

   Rust

   ```rust
   // infrastructure/persistence/postgres_adapter.rs
   impl UserRepository for PostgresUserRepository {
       fn find_by_id(&self, id: &str) -> Result<User, DomainError> {
           // Simulating a DB lookup
           let db_result: Result<UserDbModel, sqlx::Error> = sql_query_execution();

           db_result
               .map(|db_model| db_model.into()) // Success: DB Model -> Domain Model
               .map_err(|sql_err| match sql_err {
                   // Error Translation: DB Error -> Domain Error
                   sqlx::Error::RowNotFound => DomainError::UserNotFound,
                   _ => DomainError::InternalSystemError,
               })
       }
   }
   ```

3. **Presentation Layer Error Translation:** The Web Controller translates the `DomainError` received from the Use Case into an HTTP status code.
   - `DomainError::UserNotFound` ➡️ HTTP 404 Not Found
   - `DomainError::InternalSystemError` ➡️ HTTP 500 Internal Server Error

> **Rust Tip:** In practice, to reduce error translation boilerplate, utilizing the `thiserror` crate to automatically implement the `From` trait between error types is the de facto standard in the Rust backend ecosystem.

Here is the translation of the remaining content, with all formatting completely preserved.

---

## Practical Folder Structure

### Option A: Layered Structure (Package by Layer)

This is the most basic and intuitive approach. It structures folders by borrowing the Domain-Driven Design (DDD) terminology mentioned in previous steps. It is suitable for small-scale projects or in the early stages of adopting the architecture.

```plaintext
com.myapp
├── domain          // (Clean: Entities) Pure business objects
│   └── User.java
├── application     // (Clean: Use Cases) Service logic, Ports (Interfaces)
│   ├── port
│   │   ├── in      // Use Case interfaces
│   │   └── out     // Interfaces for DB access, etc.
│   └── service     // Use Case implementations
├── presentation    // (Clean: Interface Adapters - Inbound) Controllers, Views
│   ├── controller
│   └── dto
└── infrastructure  // (Clean: Interface Adapters - Outbound + Frameworks & Drivers)
    ├── persistence // DB entities, Repository implementations
    └── config      // Framework configuration files (Spring Config, etc.)
```

### Option B: Feature-Based Structure (Package by Feature)

As the system grows, dividing folders by layer like Option A decreases cohesion (e.g., to modify a User feature, you have to open all 4 folders). From the perspective of the latest trend, Modular Monolith, placing 'features' as the top-level folders and nesting the Clean Architecture layers inside them is strongly recommended as a Best Practice.

```plaintext
com.myapp
├── user/                       // [User] Domain module
│   ├── domain/                 // 1. Entities
│   ├── application/            // 2. Use Cases
│   ├── presentation/           // 3. Adapters (In)
│   └── infrastructure/         // 4. Adapters (Out) + Frameworks
│
├── order/                      // [Order] Domain module
│   ├── domain/
│   ├── application/
│   ├── presentation/
│   └── infrastructure/
│
└── common/                     // Common utilities and global configurations (Frameworks)
    └── config/
```

Using this structure, when you open the project in Finder on macOS or a text editor, you can see at a glance 'what business the app does (User, Order)' rather than 'what framework this system uses'. This is the true realization of the "Screaming Architecture" emphasized by Robert C. Martin.

---
