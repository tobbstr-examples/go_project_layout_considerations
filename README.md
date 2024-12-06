# go_project_layout_considerations

The purpose of this repository is to provide insights and code examples that can be fitted together like pieces of a puzzle to form a finished project layout. The following sections lists the different types of attributes that need to be taken into consideration when considering which project layout to use. Many of the attributes come with recommended design patterns, which are also defined.

## Use case attributes

This section categorises servers based on the complexity of their use cases. These attributes help in identifying the appropriate architectural approaches for different application scenarios. Servers can range from simple, handling a single cohesive group of use cases, to complex, managing multiple distinct groups of use cases.

| Attribute     | Description | Pattern          |
|---------------|-------------|------------------|
| simple        | The smallest kind of server with a single group of use cases. | N/A |
| complex       | Two or more groups of use cases.<br><br>An online learning platform app can be sliced in different ways. One such way is to have three groups of use cases such as course management, student management and payment. | Vertical slicing |

## API attributes

This section describes how servers expose their functionality to external clients or systems. Servers can be classified based on the number and types of APIs they provide. A server with a single API type focuses on a specific integration method, while servers with multiple API types cater to diverse client needs and communication protocols. These attributes influence the choice of architectural patterns, ensuring the server can efficiently support its intended interactions.

| Attribute     | Description                                                               | Pattern          |
|---------------|---------------------------------------------------------------------------|------------------|
| api-1         | The server has a single kind of exposed API such as gRPC, REST or SOAP | N-tier           |
| api-2+        | The server has two or more kinds of exposed APIs                       | Ports & Adapters |

## Business model attributes

This section explores the structural and behavioral characteristics of a server's business models. These attributes determine how the application's core logic interacts with data and where business rules are implemented.

- **Structural attributes** define the relationship between the server's business models and the underlying persisted models. They can range from a simple unified structure, where business and persisted models are identical, to more complex configurations like split models or event-sourced models that introduce abstraction and flexibility.

- **Behavioral attributes** describe the complexity of business logic encapsulated within the models. Models can either be rich, incorporating substantial business rules directly, or anemic, where such logic is minimal or absent.

By combining structural and behavioral attributes, a distinct set of joint attributes (structural + behavioural) can be identified, each aligning with a specific design pattern. This section outlines these combinations and recommends appropriate patterns to guide the design of robust and maintainable applications.

**Structural**

| Attribute     | Description |
|---------------|------------------|
| unified-model | A unified-model means the server's business models are directly mapped to the persisted models, with no abstraction or transformation layer between them. This setup is common in simple CRUD servers, where the same struct is used for both business logic and database operations.<br><br>Unified models are typically flat, with fields consisting primarily of primitive data types (e.g., strings, integers, or booleans). If a model includes fields that reference or depend on other models (e.g., struct A containing a collection of struct B), it is no longer a unified model—unless the database schema allows for such relationships natively, as in NoSQL databases or when using JSON data types. |
| split-model   | As opposed to the unified-model, the business models are different from the persisted models. |
| es-model      | A specialised version of the *split-model* where the application models are event-sourced |

---

**Behavioural**

| Attribute     | Description          |
|---------------|------------------|
| rich-model    | The server's business rules incorporate complex behaviour, making it advantageous to encapsulate this behavior directly within the models themselves. |
| anemic-model  | The server's business rules are trivial or non-existant. |

---

<br>
When considering the business model attributes both of these categories must be taken into consideration.
The table below shows the resulting six distinct joint attributes.

| Behavioural →<br>Structural ↓ | Rich   | Anemic |
|-------------------|----------|----------|
| **Unified**       | ur-model | ua-model |
| **Split**         | sr-model | sa-model |
| **Event-sourced** | er-model | ea-model |

The table below defines which pattern is recommended for each of the joint attributes.

| Model       | Pattern                           |
|-------------|-----------------------------------|
| ua-model    | transaction script                |
| ur-model    | active record                     |
| sa-model    | anemic domain model               |
| sr-model    | rich domain model                 |
| ea-model    | anemic event-sourced domain model |
| er-model    | rich event-sourced domain model   |

## Combinations and Recommended Patterns

This section outlines the possible combinations of use case, API, and business model attributes, along with the recommended architectural patterns for each. These combinations provide guidance on selecting the most suitable design for different server configurations.

| Combination             | Patterns                                                                      |
|-------------------------|-------------------------------------------------------------------------------|
| simple_api-1_ua-model   | N-tier + transaction script                                                   |
| simple_api-1_ur-model   | N-tier + active record                                                        |
| simple_api-1_sa-model   | N-tier + anemic domain model                                                  |
| simple_api-1_sr-model   | N-tier + rich domain model                                                    |
| simple_api-1_ea-model   | N-tier + anemic event-sourced domain model                                    |
| simple_api-1_er-model   | N-tier + rich event-sourced domain model                                      |
| simple_api-2+_ua-model  | Ports & Adapters + transaction script                                         |
| simple_api-2+_ur-model  | Ports & Adapters + active record                                              |
| simple_api-2+_sa-model  | Ports & Adapters + anemic domain model                                        |
| simple_api-2+_sr-model  | Ports & Adapters + rich domain model                                          |
| simple_api-2+_ea-model  | Ports & Adapters + anemic event-sourced domain model                          |
| simple_api-2+_er-model  | Ports & Adapters + rich event-sourced domain model                            |
| complex_api-1_ua-model  | vertical slicing + transaction script                                         |
| complex_api-1_ur-model  | vertical slicing + active record                                              |
| complex_api-1_sa-model  | vertical slicing + anemic domain model                                        |
| complex_api-1_sr-model  | vertical slicing + rich domain model                                          |
| complex_api-1_ea-model  | vertical slicing + anemic event-sourced domain model                          |
| complex_api-1_er-model  | vertical slicing + rich event-sourced domain model                            |
| complex_api-2+_ua-model | vertical slicing +<br>Ports & Adapters +<br>transaction script                |
| complex_api-2+_ur-model | vertical slicing +<br>Ports & Adapters +<br>active record                     |
| complex_api-2+_sa-model | vertical slicing +<br>Ports & Adapters +<br>anemic domain model               |
| complex_api-2+_sr-model | vertical slicing +<br>Ports & Adapters +<br>rich domain model                 |
| complex_api-2+_ea-model | vertical slicing +<br>Ports & Adapters +<br>anemic event-sourced domain model |
| complex_api-2+_er-model | vertical slicing +<br>Ports & Adapters +<br>rich event-sourced domain model   |

## Design Patterns

### Vertical Slicing

Vertical Slicing is a design pattern that organizes code by features or use cases rather than technical layers, grouping all related components (e.g., UI, business logic, and data access) into a single module. This structure makes the application’s purpose and functionality clear, simplifies maintenance, and promotes feature-focused development by reducing dependencies between unrelated areas of the system.

```
/ecommerce-app
├── /cmd                  # Entry points for the application (e.g., CLI, HTTP server)
│   └── main.go
├── /internal             # Application-specific code
│   ├── /order            # Feature: Orders                     <-- A VERTICAL SLICE
│   │   ├── handler.go    # API handler for orders
│   │   ├── service.go    # Business logic for orders
│   │   ├── repository.go # Data access for orders
│   │   ├── models.go     # Domain models for orders
│   │   └── dto.go        # Data Transfer Objects for orders
│   ├── /product          # Feature: Products                   <-- A VERTICAL SLICE
│   │   ├── handler.go    # API handler for products
│   │   ├── service.go    # Business logic for products
│   │   ├── repository.go # Data access for products
│   │   ├── models.go     # Domain models for products
│   │   └── dto.go        # Data Transfer Objects for products
│   └── /user             # Feature: Users                      <-- A VERTICAL SLICE
│       ├── handler.go    # API handler for users
│       ├── service.go    # Business logic for users
│       ├── repository.go # Data access for users
│       ├── models.go     # Domain models for users
│       └── dto.go        # Data Transfer Objects for users
└── /pkg                  # Shared libraries or utilities
    ├── db.go             # Database connection logic
    └── logger.go         # Logging utility
```

> **📌 NOTE!**
>
> If needed, it's possible to further layer a codebase so that within each vertical slice one can apply the N-tier, Ports and Adapters or any other design pattern.

### N-tier

The N-tier design pattern is a software architecture that separates an application into distinct logical layers, each with a specific responsibility, to improve modularity, scalability, and maintainability. Common layers include:

* Presentation Layer: Handles user interaction and displays information.
* Business Logic Layer: Encapsulates core application logic and rules.
* Data Access Layer: Manages interactions with databases or external storage.

Each layer interacts only with its immediate neighbor, creating a clear separation of concerns.

```
/ecommerce-app
├── /cmd                  # Entry points for the application (e.g., CLI, HTTP server)
│   └── main.go
├── /internal             # Application-specific code
│   ├── /presentation     # Handles user-facing logic (e.g., API handlers)
│   │   ├── orders_handler.go
│   │   ├── products_handler.go
│   │   └── users_handler.go
│   ├── /business         # Business logic and services
│   │   ├── orders_service.go
│   │   ├── products_service.go
│   │   └── users_service.go
│   ├── /dataaccess       # Database access logic
│   │   ├── orders_repository.go
│   │   ├── products_repository.go
│   │   └── users_repository.go
│   ├── /models           # Domain models and shared DTOs
│   │   ├── orders.go
│   │   ├── products.go
│   │   └── users.go
│   └── /utils            # General-purpose utilities
│       ├── logger.go
│       └── db.go
└── /pkg                  # Shared libraries (if needed for external reuse)
    └── validation.go
```

### Ports and Adapters

The **Ports and Adapters design pattern** (also known as **Hexagonal Architecture**) is a software architecture that separates the core business logic (the application’s domain) from external systems and technologies, enabling flexibility, testability, and maintainability. The design revolves around **Ports** and **Adapters**, with distinct roles for each.

- **Ports**:  
  - **Primary Ports**: Interfaces that represent how the application interacts with external initiators, such as user interfaces or APIs. These ports are driven by external inputs (e.g., commands from a UI or external requests).  
  - **Secondary Ports**: Interfaces that define how the application interacts with external dependencies, such as databases, message queues, or external APIs. These ports are driven by the application to perform operations like persistence or communication.

- **Adapters**: Implementations of the Ports that handle communication between the core application and external systems.  
  - For Primary Ports, Adapters might include REST controllers, gRPC handlers, or CLI tools.  
  - For Secondary Ports, Adapters might include database repositories, API clients, or messaging libraries.

By isolating the business logic behind interfaces, the pattern allows external systems to be swapped or modified independently of the core application, ensuring that the architecture remains modular and resilient to change.

```
/ecommerce-app
├── /cmd                   # Application entry points (API server, CLI, etc.)
│   └── main.go
├── /internal
│   ├── /app               # Core business logic and domain
│   │   ├── /orders
│   │   │   ├── service.go        # Business logic for orders
│   │   │   ├── models.go         # Domain models for orders
│   │   │   ├── primary_ports.go  # Interfaces for primary ports (e.g., handlers)
│   │   │   └── secondary_ports.go # Interfaces for secondary ports (e.g., repository)
│   │   ├── /products
│   │   │   ├── service.go
│   │   │   ├── models.go
│   │   │   ├── primary_ports.go
│   │   │   └── secondary_ports.go
│   │   └── /users
│   │       ├── service.go
│   │       ├── models.go
│   │       ├── primary_ports.go
│   │       └── secondary_ports.go
│   ├── /adapters          # Implementations of ports
│   │   ├── /primary       # Handlers driven by external inputs
│   │   │   ├── http
│   │   │   │   ├── orders_handler.go # REST controller for orders
│   │   │   │   ├── products_handler.go
│   │   │   │   └── users_handler.go
│   │   │   ├── grpc
│   │   │   │   ├── orders_handler.go # gRPC controller for orders
│   │   │   │   ├── products_handler.go
│   │   │   │   └── users_handler.go
│   │   ├── /secondary     # Adapters for external dependencies
│   │       ├── db
│   │       │   ├── orders_repository.go # Database repository for orders
│   │       │   ├── products_repository.go
│   │       │   └── users_repository.go
│   │       ├── messaging
│   │       │   ├── orders_publisher.go # Message queue publisher for orders
│   │       │   └── products_publisher.go
│   │       └── external_api
│   │           ├── payment_client.go   # API client for payments
│   │           └── shipping_client.go  # API client for shipping
└── /pkg                   # Shared utilities
    ├── db.go              # Database connection logic
    ├── logger.go          # Logging utility
    └── middleware.go      # Shared HTTP middleware
```

### Active Record

The **Active Record design pattern** is a data access pattern where an object serves as both a domain model and a data access layer. Each Active Record object directly maps to a database table row, with its fields corresponding to table columns, and includes methods for CRUD operations (Create, Read, Update, Delete).  

This pattern simplifies persistence logic but tightly couples the domain model to the database schema, making it most suitable for applications with simple business logic.

```go
package main

import (
	"database/sql"
	"fmt"

	_ "github.com/mattn/go-sqlite3"
)

// Active Record for the "users" table
type User struct {
	ID    int
	Name  string
	Email string
}

// Database connection
var db *sql.DB

// Create a new user in the database
func (u *User) Create() error {
	result, err := db.Exec("INSERT INTO users (name, email) VALUES (?, ?)", u.Name, u.Email)
	if err != nil {
		return err
	}
	id, _ := result.LastInsertId()
	u.ID = int(id)
	return nil
}

// Retrieve a user by ID
func (u *User) Read(id int) error {
	return db.QueryRow("SELECT id, name, email FROM users WHERE id = ?", id).Scan(&u.ID, &u.Name, &u.Email)
}

// Update the user's details
func (u *User) Update() error {
	_, err := db.Exec("UPDATE users SET name = ?, email = ? WHERE id = ?", u.Name, u.Email, u.ID)
	return err
}

// Delete the user from the database
func (u *User) Delete() error {
	_, err := db.Exec("DELETE FROM users WHERE id = ?", u.ID)
	return err
}

func main() {
	// Initialize database
	var err error
	db, err = sql.Open("sqlite3", ":memory:")
	if err != nil {
		panic(err)
	}
	defer db.Close()

	// Create table
	_, err = db.Exec("CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT, email TEXT)")
	if err != nil {
		panic(err)
	}

	// Example usage
	user := &User{Name: "Alice", Email: "alice@example.com"}
	if err := user.Create(); err != nil {
		panic(err)
	}
	fmt.Printf("Created User: %+v\n", user)

	if err := user.Read(user.ID); err != nil {
		panic(err)
	}
	fmt.Printf("Read User: %+v\n", user)

	user.Name = "Alice Smith"
	if err := user.Update(); err != nil {
		panic(err)
	}
	fmt.Printf("Updated User: %+v\n", user)

	if err := user.Delete(); err != nil {
		panic(err)
	}
	fmt.Println("Deleted User")
}
```

### Transaction Script

The **Transaction Script design pattern** is a pattern where business logic is implemented as a series of procedural steps (scripts) that execute a specific transaction. Each script typically handles a single use case or business process, directly manipulating the data and performing operations in a linear, step-by-step fashion.  

This pattern is simple and efficient for applications with straightforward business rules but can become harder to maintain as the complexity of the logic grows.

```go
package main

import (
	"fmt"
)

// Data models
type User struct {
	ID    int
	Name  string
	Email string
}

// In-memory database
var users = make(map[int]User)

// Transaction Script for creating a user
//  - The CreateUser function handles the entire logic for creating a user, including data manipulation and side effects.
//  - No domain model or service layer is involved; it's a simple procedural flow.
func CreateUser(id int, name, email string) {
	users[id] = User{ID: id, Name: name, Email: email}
	fmt.Printf("User Created: %+v\n", users[id])
}

func main() {
	// Example of using the transaction script
	CreateUser(1, "Alice", "alice@example.com")
}
```

### Anemic Domain Model

The Anemic Domain Model design pattern refers to a pattern where domain objects (models) contain only data and lack significant business logic or behavior. The business logic is typically implemented outside of the domain models, often in service layers or other components. This pattern can lead to a separation of concerns, but it often results in poor encapsulation and less cohesive code.

```go
package main

import "fmt"

// Anemic domain model for User
//  - The User struct is an Anemic Domain Model because it only contains data with no business logic or behavior.
//  - The business logic (creating a user) is handled in the transaction script (CreateUser), rather than inside the model itself.
type User struct {
	ID    int
	Name  string
	Email string
}

// Transaction script for creating a user
func CreateUser(id int, name, email string) User {
	return User{ID: id, Name: name, Email: email}
}

func main() {
	// Example of creating a user using the service
	user := CreateUser(1, "Alice", "alice@example.com")
	fmt.Printf("User Created: %+v\n", user)
}
```

### Rich Domain Model

The Rich Domain Model design pattern refers to a pattern where domain objects not only contain data but also encapsulate business logic and behavior relevant to that data. The domain models are responsible for enforcing business rules, processing data, and managing state, promoting high cohesion and encapsulation. This pattern leads to more self-contained and maintainable code, as the logic is directly tied to the domain.

```go
package main

import (
	"errors"
	"fmt"
	"strings"
)

// User represents a rich domain model with data and behavior.
//  - Encapsulation of Business Logic: The User struct contains methods like ChangeEmail and UpdateUsername that enforce business rules.
//  - High Cohesion: All logic related to the User is within the domain model itself.
//  - Behavior-Driven: The state changes occur through controlled methods, ensuring the logic is centralized and reusable.
type User struct {
	ID       int
	Username string
	Email    string
}

// ChangeEmail updates the user's email with business rules.
func (u *User) ChangeEmail(newEmail string) error {
	if !isValidEmail(newEmail) {
		return errors.New("invalid email format")
	}
	u.Email = newEmail
	return nil
}

// UpdateUsername updates the username with business rules.
func (u *User) UpdateUsername(newUsername string) error {
	if len(strings.TrimSpace(newUsername)) < 3 {
		return errors.New("username must be at least 3 characters long")
	}
	u.Username = newUsername
	return nil
}

// Helper function to validate email format (simplified).
func isValidEmail(email string) bool {
	return strings.Contains(email, "@") && strings.Contains(email, ".")
}

func main() {
	// Create a new user.
	user := User{ID: 1, Username: "alice", Email: "alice@example.com"}
	fmt.Printf("Initial User: %+v\n", user)

	// Update username using business logic.
	if err := user.UpdateUsername("ali"); err != nil {
		fmt.Println("Error updating username:", err)
	} else {
		fmt.Println("Username updated successfully!")
	}

	// Change email using business logic.
	if err := user.ChangeEmail("alice.smith@example.com"); err != nil {
		fmt.Println("Error updating email:", err)
	} else {
		fmt.Println("Email updated successfully!")
	}

	// Print final user state.
	fmt.Printf("Updated User: %+v\n", user)
}

```


### Anemic Event-Sourced Domain Model

The Anemic Event-Sourced Domain Model design pattern is a variation of the Anemic Domain Model where domain objects primarily store data, but the state changes are captured through events, typically stored in an event store. The business logic that applies to the domain objects is handled outside of the objects, often in services or application layers. This pattern uses event sourcing to track state changes, but still separates the business logic from the domain model, resulting in an anemic structure.

```go
package main

import (
	"fmt"
)

// Event structure to capture state changes
type Event struct {
	Type    string
	Payload interface{}
}

// Anemic domain model for User
//  - The User domain model remains anemic, containing only data and a collection of events (Events).
//  - The business logic (applying events) is handled outside of the model, in the service layer (ChangeUserEmail).
//  - State changes are tracked through events (in the Events slice), but the model itself doesn’t contain business behavior.
type User struct {
	ID       int
	Name     string
	Email    string
	Events   []Event // Store events to track state changes
}

// Service layer that applies events to the domain model
func (u *User) ApplyEvent(event Event) {
	switch event.Type {
	case "EmailChanged":
		u.Email = event.Payload.(string)
	}
	u.Events = append(u.Events, event)
}

// Function to change email and generate corresponding event
func ChangeUserEmail(user *User, newEmail string) {
	event := Event{
		Type:    "EmailChanged",
		Payload: newEmail,
	}
	user.ApplyEvent(event)
}

func main() {
	// Create user and apply event
	user := User{ID: 1, Name: "Alice", Email: "alice@example.com"}
	fmt.Printf("User Created: %+v\n", user)

	// Change user email via service, generating an event
	ChangeUserEmail(&user, "alice.smith@example.com")
	fmt.Printf("Updated User: %+v\n", user)
}
```

### Rich Event-Sourced Domain Model

The **Rich Event-Sourced Domain Model** design pattern combines the principles of event sourcing with a rich domain model. In this pattern, domain objects not only encapsulate business logic and behavior but also maintain their state through a series of events. These events are stored in an event store and are used to reconstruct the state of the object. The domain objects in this pattern are responsible for both managing their state transitions and applying business logic, ensuring high cohesion and encapsulation while leveraging event sourcing for tracking changes.

```go
package main

import (
	"fmt"
)

// Event structure to capture state changes
type Event struct {
	Type    string
	Payload interface{}
}

// Rich domain model for User with behavior
//  - The User domain model encapsulates business logic (ChangeEmail), and manages state changes through events.
//  - Events (Event struct) are used for event sourcing, but the domain model itself contains behavior for applying these events (ApplyEvent).
//  - The model is rich because it includes both data and behavior, ensuring that business logic is encapsulated within the model.
type User struct {
	ID       int
	Name     string
	Email    string
	Events   []Event
}

// Method to change email and store the event
func (u *User) ChangeEmail(newEmail string) {
	// Business logic: validate email
	if newEmail == "" {
		fmt.Println("Error: Email cannot be empty")
		return
	}

	// Generate event
	event := Event{
		Type:    "EmailChanged",
		Payload: newEmail,
	}
	u.ApplyEvent(event)
}

// Method to apply events to the domain model
func (u *User) ApplyEvent(event Event) {
	switch event.Type {
	case "EmailChanged":
		u.Email = event.Payload.(string)
	}
	u.Events = append(u.Events, event)
}

// Factory function to create a new user and initialize events
func NewUser(id int, name, email string) *User {
	user := &User{
		ID:     id,
		Name:   name,
		Email:  email,
		Events: []Event{},
	}
	return user
}

func main() {
	// Create a new user
	user := NewUser(1, "Alice", "alice@example.com")
	fmt.Printf("User Created: %+v\n", user)

	// Change user email using business logic inside the domain model
	user.ChangeEmail("alice.smith@example.com")
	fmt.Printf("Updated User: %+v\n", user)
}
```




The purpose of this repository is to provide insights and code examples that can be fitted together like pieces of a puzzle to form a finished project layout. The following sections lists the different types of attributes that need to be taken into consideration when considering which project layout to use. Many of the attributes come with recommended design patterns, which are also defined.

## Use case attributes

This section categorises servers based on the complexity of their use cases. These attributes help in identifying the appropriate architectural approaches for different application scenarios. Servers can range from simple, handling a single cohesive group of use cases, to complex, managing multiple distinct groups of use cases.

| Attribute     | Description | Pattern          |
|---------------|-------------|------------------|
| simple        | The smallest kind of server with a single group of use cases. | N/A |
| complex       | Two or more groups of use cases.<br><br>An online learning platform app can be sliced in different ways. One such way is to have three groups of use cases such as course management, student management and payment. | Vertical slicing |

## API attributes

This section describes how servers expose their functionality to external clients or systems. Servers can be classified based on the number and types of APIs they provide. A server with a single API type focuses on a specific integration method, while servers with multiple API types cater to diverse client needs and communication protocols. These attributes influence the choice of architectural patterns, ensuring the server can efficiently support its intended interactions.

| Attribute     | Description                                                               | Pattern          |
|---------------|---------------------------------------------------------------------------|------------------|
| api-1         | The server has a single kind of exposed API such as gRPC, REST or SOAP | N-tier           |
| api-2+        | The server has two or more kinds of exposed APIs                       | Ports & Adapters |

## Business model attributes

This section explores the structural and behavioral characteristics of a server's business models. These attributes determine how the application's core logic interacts with data and where business rules are implemented.

- **Structural attributes** define the relationship between the server's business models and the underlying persisted models. They can range from a simple unified structure, where business and persisted models are identical, to more complex configurations like split models or event-sourced models that introduce abstraction and flexibility.

- **Behavioral attributes** describe the complexity of business logic encapsulated within the models. Models can either be rich, incorporating substantial business rules directly, or anemic, where such logic is minimal or absent.

By combining structural and behavioral attributes, a distinct set of joint attributes (structural + behavioural) can be identified, each aligning with a specific design pattern. This section outlines these combinations and recommends appropriate patterns to guide the design of robust and maintainable applications.

**Structural**

| Attribute     | Description |
|---------------|------------------|
| unified-model | A unified-model means the server's business models are directly mapped to the persisted models, with no abstraction or transformation layer between them. This setup is common in simple CRUD servers, where the same struct is used for both business logic and database operations.<br><br>Unified models are typically flat, with fields consisting primarily of primitive data types (e.g., strings, integers, or booleans). If a model includes fields that reference or depend on other models (e.g., struct A containing a collection of struct B), it is no longer a unified model—unless the database schema allows for such relationships natively, as in NoSQL databases or when using JSON data types. |
| split-model   | As opposed to the unified-model, the business models are different from the persisted models. |
| es-model      | A specialised version of the *split-model* where the application models are event-sourced |

---

**Behavioural**

| Attribute     | Description          |
|---------------|------------------|
| rich-model    | The server's business rules incorporate complex behaviour, making it advantageous to encapsulate this behavior directly within the models themselves. |
| anemic-model  | The server's business rules are trivial or non-existant. |

---

<br>
When considering the business model attributes both of these categories must be taken into consideration.
The table below shows the resulting six distinct joint attributes.

| Behavioural →<br>Structural ↓ | Rich   | Anemic |
|-------------------|----------|----------|
| **Unified**       | ur-model | ua-model |
| **Split**         | sr-model | sa-model |
| **Event-sourced** | er-model | ea-model |

The table below defines which pattern is recommended for each of the joint attributes.

| Model       | Pattern                           |
|-------------|-----------------------------------|
| ua-model    | transaction script                |
| ur-model    | active record                     |
| sa-model    | anemic domain model               |
| sr-model    | rich domain model                 |
| ea-model    | anemic event-sourced domain model |
| er-model    | rich event-sourced domain model   |

## Combinations and Recommended Patterns

This section outlines the possible combinations of use case, API, and business model attributes, along with the recommended architectural patterns for each. These combinations provide guidance on selecting the most suitable design for different server configurations.

| Combination             | Patterns                                                                      |
|-------------------------|-------------------------------------------------------------------------------|
| simple_api-1_ua-model   | N-tier + transaction script                                                   |
| simple_api-1_ur-model   | N-tier + active record                                                        |
| simple_api-1_sa-model   | N-tier + anemic domain model                                                  |
| simple_api-1_sr-model   | N-tier + rich domain model                                                    |
| simple_api-1_ea-model   | N-tier + anemic event-sourced domain model                                    |
| simple_api-1_er-model   | N-tier + rich event-sourced domain model                                      |
| simple_api-2+_ua-model  | Ports & Adapters + transaction script                                         |
| simple_api-2+_ur-model  | Ports & Adapters + active record                                              |
| simple_api-2+_sa-model  | Ports & Adapters + anemic domain model                                        |
| simple_api-2+_sr-model  | Ports & Adapters + rich domain model                                          |
| simple_api-2+_ea-model  | Ports & Adapters + anemic event-sourced domain model                          |
| simple_api-2+_er-model  | Ports & Adapters + rich event-sourced domain model                            |
| complex_api-1_ua-model  | vertical slicing + transaction script                                         |
| complex_api-1_ur-model  | vertical slicing + active record                                              |
| complex_api-1_sa-model  | vertical slicing + anemic domain model                                        |
| complex_api-1_sr-model  | vertical slicing + rich domain model                                          |
| complex_api-1_ea-model  | vertical slicing + anemic event-sourced domain model                          |
| complex_api-1_er-model  | vertical slicing + rich event-sourced domain model                            |
| complex_api-2+_ua-model | vertical slicing +<br>Ports & Adapters +<br>transaction script                |
| complex_api-2+_ur-model | vertical slicing +<br>Ports & Adapters +<br>active record                     |
| complex_api-2+_sa-model | vertical slicing +<br>Ports & Adapters +<br>anemic domain model               |
| complex_api-2+_sr-model | vertical slicing +<br>Ports & Adapters +<br>rich domain model                 |
| complex_api-2+_ea-model | vertical slicing +<br>Ports & Adapters +<br>anemic event-sourced domain model |
| complex_api-2+_er-model | vertical slicing +<br>Ports & Adapters +<br>rich event-sourced domain model   |

## Design Patterns

### Vertical Slicing

Vertical Slicing is a design pattern that organizes code by features or use cases rather than technical layers, grouping all related components (e.g., UI, business logic, and data access) into a single module. This structure makes the application’s purpose and functionality clear, simplifies maintenance, and promotes feature-focused development by reducing dependencies between unrelated areas of the system.

```
/ecommerce-app
├── /cmd                  # Entry points for the application (e.g., CLI, HTTP server)
│   └── main.go
├── /internal             # Application-specific code
│   ├── /order            # Feature: Orders                     <-- A VERTICAL SLICE
│   │   ├── handler.go    # API handler for orders
│   │   ├── service.go    # Business logic for orders
│   │   ├── repository.go # Data access for orders
│   │   ├── models.go     # Domain models for orders
│   │   └── dto.go        # Data Transfer Objects for orders
│   ├── /product          # Feature: Products                   <-- A VERTICAL SLICE
│   │   ├── handler.go    # API handler for products
│   │   ├── service.go    # Business logic for products
│   │   ├── repository.go # Data access for products
│   │   ├── models.go     # Domain models for products
│   │   └── dto.go        # Data Transfer Objects for products
│   └── /user             # Feature: Users                      <-- A VERTICAL SLICE
│       ├── handler.go    # API handler for users
│       ├── service.go    # Business logic for users
│       ├── repository.go # Data access for users
│       ├── models.go     # Domain models for users
│       └── dto.go        # Data Transfer Objects for users
└── /pkg                  # Shared libraries or utilities
    ├── db.go             # Database connection logic
    └── logger.go         # Logging utility
```

> **📌 NOTE!**
>
> If needed, it's possible to further layer a codebase so that within each vertical slice one can apply the N-tier, Ports and Adapters or any other design pattern.

### N-tier

The N-tier design pattern is a software architecture that separates an application into distinct logical layers, each with a specific responsibility, to improve modularity, scalability, and maintainability. Common layers include:

* Presentation Layer: Handles user interaction and displays information.
* Business Logic Layer: Encapsulates core application logic and rules.
* Data Access Layer: Manages interactions with databases or external storage.

Each layer interacts only with its immediate neighbor, creating a clear separation of concerns.

```
/ecommerce-app
├── /cmd                  # Entry points for the application (e.g., CLI, HTTP server)
│   └── main.go
├── /internal             # Application-specific code
│   ├── /presentation     # Handles user-facing logic (e.g., API handlers)
│   │   ├── orders_handler.go
│   │   ├── products_handler.go
│   │   └── users_handler.go
│   ├── /business         # Business logic and services
│   │   ├── orders_service.go
│   │   ├── products_service.go
│   │   └── users_service.go
│   ├── /dataaccess       # Database access logic
│   │   ├── orders_repository.go
│   │   ├── products_repository.go
│   │   └── users_repository.go
│   ├── /models           # Domain models and shared DTOs
│   │   ├── orders.go
│   │   ├── products.go
│   │   └── users.go
│   └── /utils            # General-purpose utilities
│       ├── logger.go
│       └── db.go
└── /pkg                  # Shared libraries (if needed for external reuse)
    └── validation.go
```

### Ports and Adapters

The **Ports and Adapters design pattern** (also known as **Hexagonal Architecture**) is a software architecture that separates the core business logic (the application’s domain) from external systems and technologies, enabling flexibility, testability, and maintainability. The design revolves around **Ports** and **Adapters**, with distinct roles for each.

- **Ports**:  
  - **Primary Ports**: Interfaces that represent how the application interacts with external initiators, such as user interfaces or APIs. These ports are driven by external inputs (e.g., commands from a UI or external requests).  
  - **Secondary Ports**: Interfaces that define how the application interacts with external dependencies, such as databases, message queues, or external APIs. These ports are driven by the application to perform operations like persistence or communication.

- **Adapters**: Implementations of the Ports that handle communication between the core application and external systems.  
  - For Primary Ports, Adapters might include REST controllers, gRPC handlers, or CLI tools.  
  - For Secondary Ports, Adapters might include database repositories, API clients, or messaging libraries.

By isolating the business logic behind interfaces, the pattern allows external systems to be swapped or modified independently of the core application, ensuring that the architecture remains modular and resilient to change.

```
/ecommerce-app
├── /cmd                   # Application entry points (API server, CLI, etc.)
│   └── main.go
├── /internal
│   ├── /app               # Core business logic and domain
│   │   ├── /orders
│   │   │   ├── service.go        # Business logic for orders
│   │   │   ├── models.go         # Domain models for orders
│   │   │   ├── primary_ports.go  # Interfaces for primary ports (e.g., handlers)
│   │   │   └── secondary_ports.go # Interfaces for secondary ports (e.g., repository)
│   │   ├── /products
│   │   │   ├── service.go
│   │   │   ├── models.go
│   │   │   ├── primary_ports.go
│   │   │   └── secondary_ports.go
│   │   └── /users
│   │       ├── service.go
│   │       ├── models.go
│   │       ├── primary_ports.go
│   │       └── secondary_ports.go
│   ├── /adapters          # Implementations of ports
│   │   ├── /primary       # Handlers driven by external inputs
│   │   │   ├── http
│   │   │   │   ├── orders_handler.go # REST controller for orders
│   │   │   │   ├── products_handler.go
│   │   │   │   └── users_handler.go
│   │   │   ├── grpc
│   │   │   │   ├── orders_handler.go # gRPC controller for orders
│   │   │   │   ├── products_handler.go
│   │   │   │   └── users_handler.go
│   │   ├── /secondary     # Adapters for external dependencies
│   │       ├── db
│   │       │   ├── orders_repository.go # Database repository for orders
│   │       │   ├── products_repository.go
│   │       │   └── users_repository.go
│   │       ├── messaging
│   │       │   ├── orders_publisher.go # Message queue publisher for orders
│   │       │   └── products_publisher.go
│   │       └── external_api
│   │           ├── payment_client.go   # API client for payments
│   │           └── shipping_client.go  # API client for shipping
└── /pkg                   # Shared utilities
    ├── db.go              # Database connection logic
    ├── logger.go          # Logging utility
    └── middleware.go      # Shared HTTP middleware
```

### Active Record

The **Active Record design pattern** is a data access pattern where an object serves as both a domain model and a data access layer. Each Active Record object directly maps to a database table row, with its fields corresponding to table columns, and includes methods for CRUD operations (Create, Read, Update, Delete).  

This pattern simplifies persistence logic but tightly couples the domain model to the database schema, making it most suitable for applications with simple business logic.

```go
package main

import (
	"database/sql"
	"fmt"

	_ "github.com/mattn/go-sqlite3"
)

// Active Record for the "users" table
type User struct {
	ID    int
	Name  string
	Email string
}

// Database connection
var db *sql.DB

// Create a new user in the database
func (u *User) Create() error {
	result, err := db.Exec("INSERT INTO users (name, email) VALUES (?, ?)", u.Name, u.Email)
	if err != nil {
		return err
	}
	id, _ := result.LastInsertId()
	u.ID = int(id)
	return nil
}

// Retrieve a user by ID
func (u *User) Read(id int) error {
	return db.QueryRow("SELECT id, name, email FROM users WHERE id = ?", id).Scan(&u.ID, &u.Name, &u.Email)
}

// Update the user's details
func (u *User) Update() error {
	_, err := db.Exec("UPDATE users SET name = ?, email = ? WHERE id = ?", u.Name, u.Email, u.ID)
	return err
}

// Delete the user from the database
func (u *User) Delete() error {
	_, err := db.Exec("DELETE FROM users WHERE id = ?", u.ID)
	return err
}

func main() {
	// Initialize database
	var err error
	db, err = sql.Open("sqlite3", ":memory:")
	if err != nil {
		panic(err)
	}
	defer db.Close()

	// Create table
	_, err = db.Exec("CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT, email TEXT)")
	if err != nil {
		panic(err)
	}

	// Example usage
	user := &User{Name: "Alice", Email: "alice@example.com"}
	if err := user.Create(); err != nil {
		panic(err)
	}
	fmt.Printf("Created User: %+v\n", user)

	if err := user.Read(user.ID); err != nil {
		panic(err)
	}
	fmt.Printf("Read User: %+v\n", user)

	user.Name = "Alice Smith"
	if err := user.Update(); err != nil {
		panic(err)
	}
	fmt.Printf("Updated User: %+v\n", user)

	if err := user.Delete(); err != nil {
		panic(err)
	}
	fmt.Println("Deleted User")
}
```

### Transaction Script

The **Transaction Script design pattern** is a pattern where business logic is implemented as a series of procedural steps (scripts) that execute a specific transaction. Each script typically handles a single use case or business process, directly manipulating the data and performing operations in a linear, step-by-step fashion.  

This pattern is simple and efficient for applications with straightforward business rules but can become harder to maintain as the complexity of the logic grows.

```go
package main

import (
	"fmt"
)

// Data models
type User struct {
	ID    int
	Name  string
	Email string
}

// In-memory database
var users = make(map[int]User)

// Transaction Script for creating a user
//  - The CreateUser function handles the entire logic for creating a user, including data manipulation and side effects.
//  - No domain model or service layer is involved; it's a simple procedural flow.
func CreateUser(id int, name, email string) {
	users[id] = User{ID: id, Name: name, Email: email}
	fmt.Printf("User Created: %+v\n", users[id])
}

func main() {
	// Example of using the transaction script
	CreateUser(1, "Alice", "alice@example.com")
}
```

### Anemic Domain Model

The Anemic Domain Model design pattern refers to a pattern where domain objects (models) contain only data and lack significant business logic or behavior. The business logic is typically implemented outside of the domain models, often in service layers or other components. This pattern can lead to a separation of concerns, but it often results in poor encapsulation and less cohesive code.

```go
package main

import "fmt"

// Anemic domain model for User
//  - The User struct is an Anemic Domain Model because it only contains data with no business logic or behavior.
//  - The business logic (creating a user) is handled in the transaction script (CreateUser), rather than inside the model itself.
type User struct {
	ID    int
	Name  string
	Email string
}

// Transaction script for creating a user
func CreateUser(id int, name, email string) User {
	return User{ID: id, Name: name, Email: email}
}

func main() {
	// Example of creating a user using the service
	user := CreateUser(1, "Alice", "alice@example.com")
	fmt.Printf("User Created: %+v\n", user)
}
```

### Rich Domain Model

The Rich Domain Model design pattern refers to a pattern where domain objects not only contain data but also encapsulate business logic and behavior relevant to that data. The domain models are responsible for enforcing business rules, processing data, and managing state, promoting high cohesion and encapsulation. This pattern leads to more self-contained and maintainable code, as the logic is directly tied to the domain.

```go
package main

import (
	"errors"
	"fmt"
	"strings"
)

// User represents a rich domain model with data and behavior.
//  - Encapsulation of Business Logic: The User struct contains methods like ChangeEmail and UpdateUsername that enforce business rules.
//  - High Cohesion: All logic related to the User is within the domain model itself.
//  - Behavior-Driven: The state changes occur through controlled methods, ensuring the logic is centralized and reusable.
type User struct {
	ID       int
	Username string
	Email    string
}

// ChangeEmail updates the user's email with business rules.
func (u *User) ChangeEmail(newEmail string) error {
	if !isValidEmail(newEmail) {
		return errors.New("invalid email format")
	}
	u.Email = newEmail
	return nil
}

// UpdateUsername updates the username with business rules.
func (u *User) UpdateUsername(newUsername string) error {
	if len(strings.TrimSpace(newUsername)) < 3 {
		return errors.New("username must be at least 3 characters long")
	}
	u.Username = newUsername
	return nil
}

// Helper function to validate email format (simplified).
func isValidEmail(email string) bool {
	return strings.Contains(email, "@") && strings.Contains(email, ".")
}

func main() {
	// Create a new user.
	user := User{ID: 1, Username: "alice", Email: "alice@example.com"}
	fmt.Printf("Initial User: %+v\n", user)

	// Update username using business logic.
	if err := user.UpdateUsername("ali"); err != nil {
		fmt.Println("Error updating username:", err)
	} else {
		fmt.Println("Username updated successfully!")
	}

	// Change email using business logic.
	if err := user.ChangeEmail("alice.smith@example.com"); err != nil {
		fmt.Println("Error updating email:", err)
	} else {
		fmt.Println("Email updated successfully!")
	}

	// Print final user state.
	fmt.Printf("Updated User: %+v\n", user)
}

```


### Anemic Event-Sourced Domain Model

The Anemic Event-Sourced Domain Model design pattern is a variation of the Anemic Domain Model where domain objects primarily store data, but the state changes are captured through events, typically stored in an event store. The business logic that applies to the domain objects is handled outside of the objects, often in services or application layers. This pattern uses event sourcing to track state changes, but still separates the business logic from the domain model, resulting in an anemic structure.

```go
package main

import (
	"fmt"
)

// Event structure to capture state changes
type Event struct {
	Type    string
	Payload interface{}
}

// Anemic domain model for User
//  - The User domain model remains anemic, containing only data and a collection of events (Events).
//  - The business logic (applying events) is handled outside of the model, in the service layer (ChangeUserEmail).
//  - State changes are tracked through events (in the Events slice), but the model itself doesn’t contain business behavior.
type User struct {
	ID       int
	Name     string
	Email    string
	Events   []Event // Store events to track state changes
}

// Service layer that applies events to the domain model
func (u *User) ApplyEvent(event Event) {
	switch event.Type {
	case "EmailChanged":
		u.Email = event.Payload.(string)
	}
	u.Events = append(u.Events, event)
}

// Function to change email and generate corresponding event
func ChangeUserEmail(user *User, newEmail string) {
	event := Event{
		Type:    "EmailChanged",
		Payload: newEmail,
	}
	user.ApplyEvent(event)
}

func main() {
	// Create user and apply event
	user := User{ID: 1, Name: "Alice", Email: "alice@example.com"}
	fmt.Printf("User Created: %+v\n", user)

	// Change user email via service, generating an event
	ChangeUserEmail(&user, "alice.smith@example.com")
	fmt.Printf("Updated User: %+v\n", user)
}
```

### Rich Event-Sourced Domain Model

The **Rich Event-Sourced Domain Model** design pattern combines the principles of event sourcing with a rich domain model. In this pattern, domain objects not only encapsulate business logic and behavior but also maintain their state through a series of events. These events are stored in an event store and are used to reconstruct the state of the object. The domain objects in this pattern are responsible for both managing their state transitions and applying business logic, ensuring high cohesion and encapsulation while leveraging event sourcing for tracking changes.

```go
package main

import (
	"fmt"
)

// Event structure to capture state changes
type Event struct {
	Type    string
	Payload interface{}
}

// Rich domain model for User with behavior
//  - The User domain model encapsulates business logic (ChangeEmail), and manages state changes through events.
//  - Events (Event struct) are used for event sourcing, but the domain model itself contains behavior for applying these events (ApplyEvent).
//  - The model is rich because it includes both data and behavior, ensuring that business logic is encapsulated within the model.
type User struct {
	ID       int
	Name     string
	Email    string
	Events   []Event
}

// Method to change email and store the event
func (u *User) ChangeEmail(newEmail string) {
	// Business logic: validate email
	if newEmail == "" {
		fmt.Println("Error: Email cannot be empty")
		return
	}

	// Generate event
	event := Event{
		Type:    "EmailChanged",
		Payload: newEmail,
	}
	u.ApplyEvent(event)
}

// Method to apply events to the domain model
func (u *User) ApplyEvent(event Event) {
	switch event.Type {
	case "EmailChanged":
		u.Email = event.Payload.(string)
	}
	u.Events = append(u.Events, event)
}

// Factory function to create a new user and initialize events
func NewUser(id int, name, email string) *User {
	user := &User{
		ID:     id,
		Name:   name,
		Email:  email,
		Events: []Event{},
	}
	return user
}

func main() {
	// Create a new user
	user := NewUser(1, "Alice", "alice@example.com")
	fmt.Printf("User Created: %+v\n", user)

	// Change user email using business logic inside the domain model
	user.ChangeEmail("alice.smith@example.com")
	fmt.Printf("Updated User: %+v\n", user)
}
```



