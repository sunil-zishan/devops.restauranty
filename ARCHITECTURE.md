## 🍲 Architecture Overview

```mermaid
flowchart TD
    subgraph Internet[🌍 Internet]
        User[User 👤]
    end

    User -->|"HTTPS (TLS)"| Ingress[NGINX Ingress Controller 🌐\nTLS Termination]

    subgraph AppNamespace["Namespace: sunil-restauranty-dev"]
        Ingress -->|"/api/auth"| Auth[Auth Service 🔐]
        Ingress -->|"/api/discounts"| Discounts[Discounts Service 💸]
        Ingress -->|"/api/items"| Items[Items Service 🍔]
        Ingress -->|"/"| Client[React Frontend 💻]

        Auth --> MongoDB[(MongoDB 🗄️)]
        Discounts --> MongoDB
        Items --> MongoDB
    end

    subgraph Monitoring["Namespace: monitoring"]
        Prometheus[Prometheus 📊]
        Grafana[Grafana 📈]
        Blackbox[Blackbox Exporter 🔍]
    end

    %% Monitoring flows
    Blackbox --> Prometheus
    Prometheus --> Grafana

    %% Observability arrows
    Auth -.->|"Probes /health"| Blackbox
    Discounts -.->|"Probes /health"| Blackbox
    Items -.->|"Probes /health"| Blackbox
    Client -.->|"Probes /"| Blackbox
    MongoDB -.->|"Metrics"| Prometheus
