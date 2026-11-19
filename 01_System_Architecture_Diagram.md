# Online Vehicle Rental — System Architecture

**Logical architecture | Initial release: modular monolith; scale-out path shown**

---

## Architecture Diagram

```mermaid
flowchart TD
    subgraph Clients["Clients Layer"]
        CA["Customers / Admin<br/>• Web browser<br/>• Mobile browser<br/>• Admin console"]
    end

    subgraph Edge["Edge Layer"]
        EL["Edge Layer<br/>• DNS<br/>• CDN for static assets<br/>• WAF<br/>• TLS termination"]
    end

    subgraph Gateway["Gateway Layer"]
        AG["API Gateway / Proxy<br/>• Nginx / managed gateway<br/>• Rate limits<br/>• Routing"]
    end

    subgraph App["Application Layer"]
        BE["Backend Application (Node.js + Express)<br/>• Auth & User module<br/>• Vehicle & Availability module<br/>• Booking & Payment module<br/>• Notification & Admin module"]
    end

    subgraph Data["Data & Storage Layer"]
        DB[("Primary Database<br/>PostgreSQL<br/>• Users<br/>• Vehicles<br/>• Bookings<br/>• Payments<br/>• Locations")]
        CACHE[("Cache<br/>Redis<br/>• Sessions<br/>• Hot searches<br/>• Short-lived availability")]
        OBJ[("Object Storage<br/>• Vehicle photos<br/>• Invoices / documents")]
    end

    subgraph Integrations["External Services & Observability"]
        OBS["Observability<br/>• Central logs<br/>• Metrics<br/>• Traces<br/>• Alerts"]
        EXT["External Providers<br/>• Payment gateway<br/>• Email / SMS provider<br/>• Maps / geocoding"]
    end

    CA -->|HTTPS| EL
    EL --> AG
    AG -->|REST/JSON| BE

    BE -->|SQL| DB
    BE -->|cache| CACHE
    BE -->|events/API| EXT
    BE -->|logs/metrics| OBS
    EXT -->|upload| OBJ
```

---

## Component Details

### 1. Customers / Admin
- Web browser
- Mobile browser
- Admin console

### 2. Edge Layer
- DNS
- CDN for static assets
- WAF (Web Application Firewall)
- TLS termination
- Communication: Inbound traffic over **HTTPS**

### 3. API Gateway / Proxy
- Nginx / managed gateway
- Rate limits
- Routing
- Communication: Routes requests to backend via **REST/JSON**

### 4. Backend Application
- **Stack**: Node.js + Express (Modular Monolith)
- **Modules**:
  - Auth & User module
  - Vehicle & Availability module
  - Booking & Payment module
  - Notification & Admin module

### 5. Primary Database
- **Engine**: PostgreSQL
- **Data Entities**: Users, Vehicles, Bookings, Payments, Locations
- **Interface**: SQL

### 6. Cache
- **Engine**: Redis
- **Purposes**: Sessions, hot searches, short-lived availability

### 7. External Providers
- Payment gateway
- Email / SMS provider
- Maps / geocoding
- Connected via events / REST APIs

### 8. Object Storage
- Stores vehicle photos and invoices/documents
- Assets uploaded via external providers / backend

### 9. Observability
- Central logs
- Metrics
- Traces
- Alerts

---

## Architectural Principles & Policies

- **Trust Boundaries**:
  - All client traffic uses HTTPS.
  - Application secrets stay server-side.
  - Payment card data is handled directly by the payment provider (never touches application servers).
- **Scale Path**:
  - Run multiple stateless API instances behind the gateway.
  - Add read replicas, message queues, and service extraction only when justified by traffic and complexity.
