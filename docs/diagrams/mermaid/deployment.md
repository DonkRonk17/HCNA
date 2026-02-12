
```mermaid
graph TD
  subgraph Physical Hosts
    subgraph Host_A
      M1[HCNA-M1]
    end
    subgraph Host_B
      W1[HCNA-W1]
    end
  end

  subgraph Virtual/Logical
    C1[HCNA-C1 (VIP/DNS)]
  end

  M1---C1
  W1---C1
  M1==Control Link==W1
```
