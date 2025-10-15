# AI4Voice-core Architecture Diagram

## Infrastructure & Application Architecture

```mermaid
graph TB
    %% Internet Layer
    Internet["🌐 Internet Users"]
    
    %% AWS Region
    subgraph AWS["🏢 AWS Region: ap-south-1 (Mumbai)"]
        
        %% VPC
        subgraph VPC["🔒 VPC: AI4Voice-core_vpc"]
            
            %% Public Layer
            subgraph Public["🌐 Public Layer"]
                IGW["🌍 Internet Gateway"]
                ALB["⚖️ Application Load Balancer"]
                Bastion["🖥️ Bastion Host"]
            end
            
            %% Private Layer
            subgraph Private["🔐 Private Layer"]
                subgraph EKS["☸️ EKS Cluster: AI4Voice-core"]
                    ControlPlane["🎛️ Control Plane<br/>(Private Endpoint)"]
                    WorkerNodes["🖥️ Worker Nodes<br/>(t3a.large)"]
                end
                
                subgraph Applications["📦 Applications"]
                    AIModels["🤖 AI Models<br/>(3 Pods)"]
                    DhruvaApp["🎵 Dhruva App<br/>(2 Pods)"]
                end
            end
            
            %% Services Layer
            subgraph Services["🔧 Services"]
                AIService["🤖 AI Service"]
                DhruvaService["🎵 Dhruva Service"]
                Ingress["🌐 Ingress Controller"]
            end
        end
    end
    
    %% External Services
    subgraph External["🌍 External Services"]
        ECR["📦 ECR Registry"]
        CloudWatch["📊 CloudWatch"]
    end
    
    %% Traffic Flow
    Internet --> IGW
    IGW --> ALB
    ALB --> Ingress
    Ingress --> AIService
    Ingress --> DhruvaService
    AIService --> AIModels
    DhruvaService --> DhruvaApp
    
    %% Management
    Bastion --> ControlPlane
    ControlPlane --> WorkerNodes
    WorkerNodes --> AIModels
    WorkerNodes --> DhruvaApp
    
    %% External Access
    WorkerNodes --> ECR
    ControlPlane --> CloudWatch
    
    %% Styling
    classDef internet fill:#e3f2fd,stroke:#1976d2,stroke-width:3px
    classDef public fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef private fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef applications fill:#fce4ec,stroke:#c2185b,stroke-width:2px
    classDef services fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef external fill:#e0f2f1,stroke:#00695c,stroke-width:2px
    
    class Internet internet
    class Public,IGW,ALB,Bastion public
    class Private,EKS,ControlPlane,WorkerNodes private
    class Applications,AIModels,DhruvaApp applications
    class Services,AIService,DhruvaService,Ingress services
    class External,ECR,CloudWatch external
```

## Network Flow Diagram

```mermaid
sequenceDiagram
    participant User as 👤 End User
    participant ALB as ⚖️ Load Balancer
    participant Ingress as 🌐 Ingress
    participant Service as 🔧 Service
    participant Pod as 📦 App Pod
    
    User->>ALB: 1. HTTP Request
    ALB->>Ingress: 2. Route Request
    Ingress->>Service: 3. Forward to Service
    Service->>Pod: 4. Load Balance to Pod
    Pod->>Service: 5. Process & Respond
    Service->>Ingress: 6. Return Response
    Ingress->>ALB: 7. Forward Response
    ALB->>User: 8. HTTP Response
```

## Security Architecture

```mermaid
graph TB
    subgraph Security["🔒 Security Layers"]
        Network["🌐 Network Security<br/>• Security Groups<br/>• Private Subnets<br/>• NAT Gateway"]
        Access["🔑 Access Control<br/>• IAM Roles<br/>• Bastion Host<br/>• Private EKS"]
        Data["💾 Data Security<br/>• Encryption at Rest<br/>• Encryption in Transit<br/>• Kubernetes Secrets"]
    end
    
    subgraph Monitoring["📊 Monitoring"]
        CloudWatch["📊 CloudWatch<br/>Logs & Metrics"]
    end
    
    Security --> Monitoring
```

## Application Deployment Architecture

```mermaid
graph TB
    subgraph Apps["🚀 Applications"]
        AI["🤖 AI Models<br/>• 3 Pods<br/>• Model Inference<br/>• GPU/CPU Optimized"]
        Dhruva["🎵 Dhruva App<br/>• 2 Pods<br/>• Voice Processing<br/>• Web Interface"]
    end
    
    subgraph Infrastructure["🏗️ Infrastructure"]
        Storage["💾 Storage<br/>• EBS Volumes<br/>• 100GB per Node"]
        Config["⚙️ Configuration<br/>• ConfigMaps<br/>• Secrets"]
        Services["🔧 Services<br/>• Load Balancing<br/>• Service Discovery"]
    end
    
    AI --> Services
    Dhruva --> Services
    AI --> Storage
    Dhruva --> Storage
    AI --> Config
    Dhruva --> Config
```

## Resource Allocation

| Component | CPU | Memory | Storage | Replicas | Purpose |
|-----------|-----|--------|---------|----------|---------|
| AI Models | 1000m | 4Gi | 100GB | 3 | Model Inference |
| Dhruva App | 500m | 2Gi | 100GB | 2 | Application Logic |
| EKS Nodes | 2000m | 8Gi | 100GB | 2 | Worker Nodes |
| ALB | - | - | - | 2 | Load Balancing |

## Endpoints

- **AI Models**: `https://ai-models.ai4voice.com`
- **Dhruva App**: `https://dhruva.ai4voice.com`
- **EKS API**: Private endpoint (via bastion host)
- **Monitoring**: `https://monitoring.ai4voice.com`
