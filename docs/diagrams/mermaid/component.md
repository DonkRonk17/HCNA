
```mermaid
flowchart LR
  subgraph Clients
    U[Users/Apps]
  end

  subgraph Composite_Plane
    C1[Composite Node C1]
  end

  subgraph Control_Plane
    M1[Moderator M1]
  end

  subgraph Data_Plane
    W1[Worker W1]
  end

  U --> C1
  C1 <-. Control Bus .-> M1
  M1 -- Intents/Policy --> W1
  W1 -- Results/Telemetry --> M1
  W1 --> C1
```
