# AI4Voice-core Architecture Diagram

## Infrastructure & Application Architecture

```mermaid
graph TB
    %% Internet Layer
    Internet["🌐 Internet Users"]
    
    %% AWS Region
    subgraph AWS["🏢 AWS Region: ap-south-1 (Mumbai)"]
        
        %% VPC
        subgraph VPC["🔒 VPC: AI4Voice-core_vpc (30.0.0.0/16)"]
            
            %% Public Subnets
            subgraph PublicSubnets["🌐 PUBLIC SUBNETS"]
                subgraph PublicAZ1["📍 Public AZ-1a (30.0.8.0/22)"]
                    IGW["🌍 Internet Gateway"]
                    NAT1["🔀 NAT Gateway"]
                    ALB["⚖️ Application Load Balancer"]
                    Bastion1["🖥️ Bastion Host"]
                end
                
                subgraph PublicAZ2["📍 Public AZ-1b (30.0.12.0/22)"]
                    NAT2["🔀 NAT Gateway"]
                    Bastion2["🖥️ Bastion Host"]
                end
            end
            
            %% Private Subnets
            subgraph PrivateSubnets["🔐 PRIVATE SUBNETS"]
                subgraph PrivateAZ1["📍 Private AZ-1a (30.0.0.0/22)"]
                    EKSControl1["☸️ EKS Control Plane"]
                    
                    subgraph CPUWorker1["🖥️ CPU Worker Node (t3a.xlarge)"]
                        DhruvaPod1["🎵 Dhruva App Pod 1"]
                        DhruvaPod2["🎵 Dhruva App Pod 2"]
                    end
                    
                    subgraph GPUWorker1["🚀 GPU Worker Node (g4dn.xlarge)"]
                        AIModel1["🤖 AI Model Pod 1"]
                        AIModel2["🤖 AI Model Pod 2"]
                    end
                end
                
                subgraph PrivateAZ2["📍 Private AZ-1b (30.0.4.0/22)"]
                    EKSControl2["☸️ EKS Control Plane"]
                    
                    subgraph GPUWorker2["🚀 GPU Worker Node (g4dn.xlarge)"]
                        AIModel3["🤖 AI Model Pod 3"]
                    end
                end
            end
            
            %% EKS Cluster
            subgraph EKSCluster["☸️ EKS Cluster: AI4Voice-core"]
                EKSControl1
                EKSControl2
                CPUWorker1
                GPUWorker1
                GPUWorker2
            end
            
            %% Services Layer
            subgraph Services["🔧 Kubernetes Services"]
                AIService["🤖 AI Model Service"]
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
    AIService --> AIModel1
    AIService --> AIModel2
    AIService --> AIModel3
    DhruvaService --> DhruvaPod1
    DhruvaService --> DhruvaPod2
    
    %% Management
    Bastion1 --> EKSControl1
    Bastion2 --> EKSControl2
    EKSControl1 --> CPUWorker1
    EKSControl1 --> GPUWorker1
    EKSControl2 --> GPUWorker2
    
    %% External Access
    CPUWorker1 --> ECR
    GPUWorker1 --> ECR
    GPUWorker2 --> ECR
    EKSControl1 --> CloudWatch
    EKSControl2 --> CloudWatch
    
    %% NAT Gateway Access
    CPUWorker1 --> NAT1
    GPUWorker1 --> NAT1
    GPUWorker2 --> NAT2
    
    %% Styling
    classDef internet fill:#e3f2fd,stroke:#1976d2,stroke-width:3px
    classDef public fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef private fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef cpu fill:#e1f5fe,stroke:#0277bd,stroke-width:2px
    classDef gpu fill:#fce4ec,stroke:#c2185b,stroke-width:2px
    classDef applications fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef services fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    classDef external fill:#e0f2f1,stroke:#00695c,stroke-width:2px
    
    class Internet internet
    class PublicSubnets,PublicAZ1,PublicAZ2,IGW,NAT1,NAT2,ALB,Bastion1,Bastion2 public
    class PrivateSubnets,PrivateAZ1,PrivateAZ2,EKSControl1,EKSControl2 private
    class CPUWorker1,DhruvaPod1,DhruvaPod2 cpu
    class GPUWorker1,GPUWorker2,AIModel1,AIModel2,AIModel3 gpu
    class EKSCluster services
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

| Component | Instance Type | CPU | Memory | Storage | Replicas | Purpose |
|-----------|---------------|-----|--------|---------|----------|---------|
| **CPU Worker Nodes** | t3a.xlarge | 4 vCPU | 16Gi | 100GB | 1 | Dhruva App |
| **GPU Worker Nodes** | g4dn.xlarge | 4 vCPU | 16Gi | 100GB | 2 | AI Models |
| **AI Models** | - | 2000m | 8Gi | - | 3 | Model Inference |
| **Dhruva App** | - | 1000m | 4Gi | - | 2 | Application Logic |
| **EKS Control Plane** | - | - | - | - | 2 | Cluster Management |
| **ALB** | - | - | - | - | 1 | Load Balancing |
| **NAT Gateway** | - | - | - | - | 2 | Internet Access |
| **Bastion Host** | t3a.medium | 2 vCPU | 4Gi | 20GB | 2 | Management Access |

## Subnet Breakdown

### 🌐 **Public Subnets**
| Subnet | AZ | CIDR | Components | Purpose |
|--------|----|----- |------------|---------|
| **Public AZ-1a** | ap-south-1a | 30.0.8.0/22 | IGW, NAT Gateway, ALB, Bastion Host | Internet access, Load balancing |
| **Public AZ-1b** | ap-south-1b | 30.0.12.0/22 | NAT Gateway, Bastion Host | High availability, Management access |

### 🔐 **Private Subnets**
| Subnet | AZ | CIDR | Components | Purpose |
|--------|----|----- |------------|---------|
| **Private AZ-1a** | ap-south-1a | 30.0.0.0/22 | EKS Control Plane, CPU Worker (t3a.xlarge), GPU Worker (g4dn.xlarge) | Dhruva App, AI Models |
| **Private AZ-1b** | ap-south-1b | 30.0.4.0/22 | EKS Control Plane, GPU Worker (g4dn.xlarge) | AI Models, High availability |

### 🔀 **Network Components**
- **Internet Gateway**: Provides internet access to public subnets
- **NAT Gateway**: Allows private subnets to access internet for outbound traffic
- **Route Tables**: Direct traffic between subnets and gateways
- **Security Groups**: Control inbound/outbound traffic to resources

## Endpoints

- **AI Models**: `https://ai-models.ai4voice.com`
- **Dhruva App**: `https://dhruva.ai4voice.com`
- **EKS API**: Private endpoint (via bastion host)
- **Monitoring**: `https://monitoring.ai4voice.com`
