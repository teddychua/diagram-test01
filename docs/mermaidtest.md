





```mermaid

classDiagram

class UserController {
    <<Class>>
    -ICreateUserService _createUserService
    +UserController(ICreateUserService createUserService)
    +Create(string email, string username) User
}

class CreateUserRequest {
   +string Email
   +string Username

   +Validate() bool
}

class ICreateUserService {
    <<Interface>>
    +Call(CreateUserRequest createUserRequest) User
}

class CreateUserService {
    <<Class>>
    -UserModel _userModel
    -SendWelcomeEmailService _sendWelcomeEmailService
    -Kafka _kafka
    +CreateUserService(UserModel um, SendWelcomeEmailService es, Kafka k)
    +Call(CreateUserRequest createUserRequest) User
    -CheckActiveUsers(List~User~ users) bool
}



    class UserModel {
        <<Class>>
        +FindUsersByEmail(string email) List~User~
        +CreateUser(string email, string username) User
    }

    class User {
        <<Class>>
        +int UserId
        +string Username
        +string Email
    }

    class SendWelcomeEmailService {
        <<Class>>
        +Call(User user) bool
    }

    class Kafka {
        <<Class>>
        +PublishUserCreatedEvent(User User) bool
    }

    UserController ..> ICreateUserService: depends on
    CreateUserService ..|> ICreateUserService: implements
    UserController ..> CreateUserRequest: depends on
    CreateUserService ..> UserModel: depends on
    UserModel ..> User: depends on
    CreateUserService ..> SendWelcomeEmailService: depends on
    CreateUserService ..> Kafka: depends on

```



```


```





```mermaid

---
title: POST /users Request Handling
---
sequenceDiagram
    autonumber
    
    participant UserController
    participant CreateUserService
    participant UserModel
    participant SendWelcomeEmailService
    participant Kafka

    UserController->>+CreateUserService: call
    CreateUserService->>UserModel: find_users_by_email
    UserModel-->>CreateUserService: array of Users

    loop
        CreateUserService->>CreateUserService: check_active_users
    end

    CreateUserService->>UserModel: create_user
    UserModel-->>CreateUserService: User

    par
        CreateUserService->>SendWelcomeEmailService: send_welcome_email
        SendWelcomeEmailService-->>CreateUserService: boolean

        CreateUserService->>Kafka: publish_user_created_event
        Kafka-->>CreateUserService: boolean
    end
    
    CreateUserService-->>-UserController: User

```



```


```

```mermaid

---
title: Streamy Entity Relationship Diagram
---
erDiagram
    TITLE {
        int title_id PK
        int type_id FK
        string name
        datetime release_date
    }

    TITLE_TYPE {
        int type_id PK
        string type
    }

    ACTOR {
        int actor_id PK
        string name
        date date_of_birth
    }

    TITLE_ACTOR {
        int title_id PK "FK"
        int actor_id PK "FK"
    }

    GENRE {
        int genre_id PK
        string name
    }

    TITLE_GENRE {
        int title_id PK "FK"
        int genre_id PK "FK"
    }

    EPISODE {
        int episode_id PK
        int season_id FK
        string name
        int season_number
        int episode_number
        datetime release_date
    }

    SEASON {
        int season_id PK 
        int title_id FK
        int season_number
        date release_year
    }

    REVIEW {
        int review_id PK
        int title_id FK
        int episode_id FK
        int season_id FK
        string review_by
        datetime review_date
        string review_text
    }

    TITLE }|..|| TITLE_TYPE: has
    TITLE ||--o{ TITLE_GENRE: "belongs to"
    TITLE ||--|{ TITLE_ACTOR: features
    TITLE ||..|{ SEASON: contains

    TITLE_GENRE }o--|| GENRE: references

    TITLE_ACTOR }|--|| ACTOR: references

    EPISODE }|..|| SEASON: contains

    REVIEW }o..o| TITLE: "made against"
    REVIEW }o..o| EPISODE: "made against"
    REVIEW }o..o| SEASON: "made against"


```






```


```


```mermaid

---
title: "Listing Service C4 Model: Component Diagram"
---
flowchart TD
classDef container fill:#1168bd,stroke:#0b4884,color:#ffffff
classDef externalSystem fill:#666,stroke:#0b4884,color:#ffffff
classDef component fill:#85bbf0,stroke:#5d82a8,color:#000000

Browser["Browser
[Web Browser]

Used by a user to browse
the website"]

MA["Application
[Xamarin Application]

Allows members to view and review
titles from their mobile devices"]

R["In-Memory Cache
[Redis]

Titles and their reviews
are cached"] 

K["Message Broker
[Kafka]

Important domain events
are published to Kafka"]

TS["Title Service
[Software System]

Provides an API to retrieve
title information"]

RS["Review Service
[Software System]

Provides an API to retrieve
and submit reviews"]

SS["Search Service
[Software System]

Provides an API to search
for titles"]

TCont["Title Controller
[ASP.NET MVC Controller]

Allows users to view details
about titles"]

SCont["Search Controller
[ASP.NET MVC Controller]

Allows users to search
for titles"]

RCont["Review Controller
[ASP.NET MVC Controller]

Allows users to read and
write reviews"]

TComp["Title Component
[ASP.NET Namespace]

Provides information on titles,
retrieves information from the title service
and caches titles"]

SComp["Search Component
[ASP.NET Namespace]

Searches titles using the
search service"]

RComp["Review Component
[ASP.NET Namespace]

Provides review information,
submits new reviews
and publishes domain events"]

Browser-- "Submits requests to\n[HTTPS]" --->TCont
MA-- "Submits requests to\n[HTTPS]" --->TCont

MA-- "Submits requests to\n[HTTPS]" --->SCont
Browser-- "Submits requests to\n[HTTPS]" --->SCont

MA-- "Submits requests to\n[HTTPS]" --->RCont
Browser-- "Submits requests to\n[HTTPS]" --->RCont

subgraph listing-service[Listing Service]
   TCont--->TComp
   RCont--->TComp
   RCont--->RComp

   SCont--->SComp
end

TComp--->TS
TComp--->R

RComp--->R
RComp--->K
RComp--->RS

SComp--->SS

class MA,R container
class SS,RS,TS,K,Browser externalSystem
class RComp,SComp,TComp,RCont,SCont,TCont component
style listing-service fill:none,stroke:#CCC,stroke-width:2px
style listing-service color:#fff,stroke-dasharray: 5 5

```

```


```

```mermaid

flowchart TD

User["Premium Member\n[Person]\nA user of the website who has
purchased a subscription"]

WA["Web Application\n[.NET Core MVC Application]\nAllows members to view\nand review titles from a web browser.\nAlso exposes an API for the mobile app"]

MA["Mobile Application
[Xamarin Application]
Allows members to view and review
titles from their mobile devices"]

R[("In-Memory Cache
[Redis]
Titles and their reviews
are cached")]

K["Message Broker
[Kafka]
Important domain events
are published to Kafka"]
TS["Title Service
[Software System]
Provides an API to retrieve
title information"]
RS["Reviews Service
[Software System]
Provides an API to retrieve
and submit reviews"]
SS["Search Service
[Software System]
Provides an API to search
for titles"]


User-- "Views titles, searches titles
and reviews titles using
[HTTPS]" -->WA
User-- "Views titles, searches titles
and reviews titles using
[HTTPS]" -->MA
WA-. "Publishes messages to\n[Binary over TCP]" ..->K
WA-- "Makes API calls to\n[HTTPS]" --->TS
WA-- "Makes API calls to\n[HTTPS]" --->RS
WA-- "Makes API calls to\n[HTTPS]" --->SS


subgraph listing-service[Listing Service]
   MA-- "Makes API calls to\n[HTTPS]" -->WA
   WA-- "Reads and writes to\n[REdis Serialization Protocol]" -->R
end


classDef container fill:#1168bd,stroke:#0b4884,color:#ffffff
classDef person fill:#08427b,stroke:#052e56,color:#ffffff
classDef supportingSystem fill:#666,stroke:#0b4884,color:#ffffff
class User person
class WA,MA container
class TS,RS,SS,K supportingSystem
style listing-service fill:none,stroke:#CCC,stroke-width:2px
style listing-service color:#fff,stroke-dasharray: 5 5


```


```


```


```mermaid
---
title: "Listing Service C4 Model: System Context"
---

flowchart TD
User["Premium Member\n[Person]\n\nA user of the website who has purchased a subscription\n\n"]

LS["Listings Service\n[Software System]\n\nServes web pages displaying title listings to the end users\n\n"]

User -- "Views titles, searches titles\nand reviews titles using" --> LS

TS["Title Service\n[Software System]\n\nProvides an API to retrieve title information\n\n"]
RS["Review Service\n[Software System]\n\nProvides an API to retrieve and submit reviews\n\n"]
SS["Search Service\n[Software System]\n\nProvides an API to search for titles\n\n"]

LS-- "Retrieves title information from" --> TS
LS-- "Retrieves from and submits reviews to" --> RS
LS-- "Searches for titles using" --> SS

classDef focusSystem fill:#1168bd,stroke:#0b4884,color:#ffffff
classDef supportingSystem fill:#666,stroke:#0b4884,color:#ffffff
classDef person fill:#08427b,stroke:#052e56,color:#ffffff
class User person
class LS focusSystem
class TS,RS,SS supportingSystem

```

```


```

```mermaid
---
title: User Signup Flow
---

sequenceDiagram
autonumber
actor Browser
participant SUS as Sign Up Service
participant US as User Service
links US: {"Repository": "https://www.example.com", "TOC": "https://www.example.com/toc"}
participant Kafka
participant Email as Email Service
%%participant Kafka


Browser ->> SUS: GET /sign_up
activate SUS
SUS -->> Browser: 200 OK (HTML page)
deactivate SUS


Browser ->> SUS: POST /sign_up
activate SUS
SUS ->> SUS: Validate input

alt invalid input
   SUS -->> Browser: Error
else valid input
   SUS ->> US: POST /users
   activate US
   US ->> Kafka: Queue Test Message
   Note left of Kafka: other services tke action based on this event
   US -->> SUS: 201 Created (User)
   deactivate US
   SUS -->> Browser: 301 Redirect (Login page)
end
deactivate SUS


Email -) Kafka: Test





```

```


```

```mermaid
---
title: Stream Domain Model example
---

classDiagram
   Title "1..*" -- "1..*" Genre: is associated with
   Title "1" *-- "0..*" Season: has
   Title *-- Review: has
   Title o-- Actor: features
   link Title "https://www.example.com" _blank 

   Viewer --> Title: watches

   Season "1" *-- "1..*" Episode: contains
   Season "1" *-- "0..*" Review: has

   Episode *-- Review: has

   TVShow --|> Title: implements
   Short --|> Title: implements
   Film --|> Title: implements

```


```


```


```mermaid
flowchart LR
a-->b & c-->d
```

> test


# test

*blah*

- this is 
- that is 

`my code goes here`

```
this is also clode
blah blah
```
