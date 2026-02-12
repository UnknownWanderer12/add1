# add1


flowchart LR
  subgraph AWS_EC2["AWS EC2 (Docker host, Security Group)"]
    subgraph DockerNet["Docker Bridge Network"]
      GW[API Gateway]
      PROD[Product API<br/>MySQL]
      CAT[Category API<br/>MongoDB]
      CFG[Config Server]
      VAULT[Vault Server]
      KAFKA[Kafka Broker]
      ZK[Zookeeper]
      KC[Keycloak]

      GW -->|JWT| PROD
      GW -->|JWT| CAT
      KC --> GW
      CFG --> PROD
      CFG --> CAT
      VAULT --> PROD
      VAULT --> CAT
      CAT -->|CategoryEvent| KAFKA
      KAFKA -->|consume| PROD
      PROD <-->|JDBC| MySQL["(MySQL)"]
      CAT <-->|Driver| Mongo["(MongoDB)"]
    end
  end

-------------------------------------------------------------------------------------------------------------------------


  ---
config:
  theme: neo-dark
---
flowchart LR
 subgraph User["User"]
        C["Client / Browser"]
  end
 subgraph DockerHub["Docker Hub Registry"]
        DH["(unknownwanderer/cognizantmsjan2026:productapiv1)"]
  end
 subgraph CI["Build & Push"]
        Code["Source Code<br>(API)"]
        Dockerfile["Dockerfile"]
        Build(["docker build"])
        Tag(["docker tag"])
        Push(["docker push"])
  end
 subgraph Deploy["Deployment: productdeploy"]
    direction LR
        Pod1{{"Pod: productapp"}}
        Pod2{{"Pod: productapp"}}
  end
 subgraph EKS["Kubernetes Cluster (EKS)"]
    direction TB
        LB["(Service: LoadBalancer<br>productservice:7074)"]
        Deploy
        MySQL["(Service: ClusterIP<br>mysql:3306)"]
        DB["(MySQL Pod/StatefulSet)"]
  end
 subgraph Scaling["Autoscaling"]
    direction TB
        HPA["HPA: Horizontal Pod Autoscaler<br>min=1, max=5, cpu=50%"]
        CA["Cluster Autoscaler (EKS Nodes)"]
  end
    Code --> Dockerfile
    Dockerfile --> Build
    Build --> Tag
    Tag --> Push
    Push --> DH
    LB --> Pod1 & Pod2
    Pod1 -- env MYSQL_URL --> MySQL
    Pod2 -- env MYSQL_URL --> MySQL
    MySQL --> DB
    DH -. image pull .-> Pod1 & Pod2
    C -- HTTP :7074 --> LB
    HPA -. scales pods .-> Deploy
    CA -. adds/removes nodes .-> EKS
    n1["Text Block"]

    n1@{ shape: text}
     C:::ext
     DH:::ci
     Code:::ci
     Dockerfile:::ci
     Build:::ci
     Tag:::ci
     Push:::ci
     Pod1:::k8s
     Pod1:::k8s
     Pod2:::k8s
     Pod2:::k8s
     LB:::svc
     LB:::svc
     MySQL:::svc
     MySQL:::svc
     DB:::k8s
     DB:::k8s
    classDef k8s fill:#E3F2FD,stroke:#90CAF9,stroke-width:1px
    classDef ext fill:#FFF3E0,stroke:#FFB74D,stroke-width:1px
    classDef svc fill:#E8F5E9,stroke:#81C784,stroke-width:1px
    classDef ci fill:#F3E5F5,stroke:#BA68C8,stroke-width:1px
    style C fill:#000000
    style DH fill:#000000
    style Code fill:#000000
    style Dockerfile fill:#000000
    style Build stroke:#000000,fill:#000000
    style Tag fill:#000000
    style Push fill:#000000
    style Pod1 fill:#000000
    style Pod2 fill:#000000
    style LB fill:#000000
    style MySQL fill:#000000
    style DB fill:#000000
    style n1 fill:#000000

  Client((Client)) -->|HTTPS| GW
