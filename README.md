# System Diagrams

This file contains all the academic system models for ProjectEngine. You can use a Markdown previewer (like the built-in VS Code preview or GitHub) to view the rendered diagrams.

---

## 1. Context Diagram
The Context Diagram represents the entire system as a single black-box process, demonstrating how external entities (Clients and Developers) interact with the system at the highest level.

*(Note: In traditional systems analysis, the Context Diagram and DFD Level 0 are often synonymous. This represents the absolute highest-level view).*

```mermaid
flowchart LR
    Client[Client]
    Dev[Developer]
    Sys((ProjectEngine\nSystem))

    Client -- "Credentials, Project Postings, Chat" --> Sys
    Sys -- "Developer Profiles, Dashboard Stats" --> Client

    Dev -- "Credentials, Skills, Applications" --> Sys
    Sys -- "Job Listings, Workspaces" --> Dev
```

---

## 2. ERD (Chen's Notation)
Chen's notation emphasizes Entities (rectangles) and Relationships (diamonds). Attributes (ovals) are simplified here for readability. 

```mermaid
flowchart TD
    %% Entities (Rectangles)
    U[Users]
    C[Clients]
    D[Developers]
    P[Projects]
    S[Skills]
    M[Workspace_Messages]
    T[Workspace_Tasks]

    %% Relationships (Diamonds)
    IsA1{Is A}
    IsA2{Is A}
    Posts{Posts}
    Applies{Applies to}
    HasSkill{Has Skill}
    HasMsg{Contains}
    HasTask{Tracks}

    %% Connect Entities to Relationships
    U ---|1| IsA1 ---|0..1| C
    U ---|1| IsA2 ---|0..1| D
    
    C ---|1| Posts ---|N| P
    D ---|1| Applies ---|N| P
    D ---|M| HasSkill ---|N| S
    
    P ---|1| HasMsg ---|N| M
    U ---|1| HasMsg
    
    P ---|1| HasTask ---|N| T

    %% Key Attributes (Ovals)
    u_id([user_id]) --- U
    c_id([client_id]) --- C
    d_id([dev_id]) --- D
    p_id([project_id]) --- P
    s_id([skill_name]) --- S
```

---

## 3. ERD (Database Schema)
This is the physical database schema using standard Crow's Foot notation. It maps exactly to the SQL Server database, showing primary keys (PK), foreign keys (FK), and column data types.

```mermaid
erDiagram
    Users ||--o| Clients : "role = Client"
    Users ||--o| Developers : "role = Developer"
    Developers ||--o{ Developer_Skills : "has"
    Skills ||--o{ Developer_Skills : "tagged in"
    Clients ||--o{ Projects : "posts"
    Projects ||--o{ Project_Applications : "receives"
    Developers ||--o{ Project_Applications : "submits"
    Projects ||--o{ Workspace_Messages : "contains"
    Users ||--o{ Workspace_Messages : "sends"
    Projects ||--o{ Workspace_Tasks : "tracks"

    Users {
        int user_id PK
        nvarchar email UK
        nvarchar password_hash
        nvarchar role
    }
    Clients {
        int client_id PK,FK
        nvarchar company_name
    }
    Developers {
        int dev_id PK,FK
        nvarchar full_name
        nvarchar level
        bit is_booked
    }
    Skills {
        int skill_id PK
        nvarchar skill_name UK
    }
    Developer_Skills {
        int dev_id PK,FK
        int skill_id PK,FK
    }
    Projects {
        int project_id PK
        int client_id FK
        nvarchar title
        nvarchar status
    }
    Project_Applications {
        int application_id PK
        int project_id FK
        int dev_id FK
        nvarchar status
    }
    Workspace_Messages {
        int message_id PK
        int project_id FK
        int sender_user_id FK
        nvarchar message_body
    }
    Workspace_Tasks {
        int task_id PK
        int project_id FK
        nvarchar title
        nvarchar status
    }
```

---

## 4. DFD Level 0
The Data Flow Diagram Level 0 breaks the Context Diagram open slightly to show data stores at a very high level, maintaining the single main process.

```mermaid
flowchart LR
    Client[Client]
    Dev[Developer]
    
    Sys((0.0 ProjectEngine Platform))
    
    DB[(Main Database)]

    Client -- "Account Info, Project Details, Messages" --> Sys
    Sys -- "Dashboard Views, Dev Profiles, Workspace State" --> Client

    Dev -- "Profile Data, Applications, Task Updates" --> Sys
    Sys -- "Job Listings, Hire Approvals, Workspace State" --> Dev
    
    Sys <--> |"SQL Read/Write"| DB
```

---

## 5. DFD Level 1
Level 1 Process Decomposition breaks the main system down into its core subsystems (Processes 1.0, 2.0, 3.0) and shows how they route data to specific logical Data Stores.

```mermaid
flowchart TB
    %% External Entities
    C[Client]
    D[Developer]

    %% Processes
    P1((1.0 Manage Identity))
    P2((2.0 Manage Projects))
    P3((3.0 Workspace Sync))

    %% Data Stores
    D1[(D1: Auth & Profiles)]
    D2[(D2: Projects & Apps)]
    D3[(D3: Tasks & Chat)]

    %% Entity to Process Flows
    C -- "Login / Profile Updates" --> P1
    C -- "Post Project / Review Apps" --> P2
    C -- "Read Tasks / Send Chat" --> P3

    D -- "Login / Skill Updates" --> P1
    D -- "Apply to Project" --> P2
    D -- "Move Tasks / Send Chat" --> P3

    %% Process to Data Store Flows
    P1 <--> |"Verify / Save Users"| D1
    P2 <--> |"Save / Read Projects"| D2
    P3 <--> |"Save / Read Board"| D3

    %% Cross-Process Communication
    P1 -. "Auth Token" .-> P2
    P1 -. "Auth Token" .-> P3
    P2 -. "Active Project Auth" .-> P3
```
