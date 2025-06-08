# Task 2: Implement Functional Modules

Build the server incrementally, adding features until it matches the capabilities of the existing ChirpStack Network Management Server.

## Phase 1 – Core Services
1. **Gateway Communication**
   - Implement a protocol-agnostic interface for gateways (e.g. MQTT in the reference design).
   - Handle uplink reception and buffering for deduplication.
2. **Device Management**
   - Create CRUD operations for devices and store runtime sessions.
   - Support activation via a Join Server through a pluggable backend interface.
3. **Downlink Scheduling**
   - Provide a queue for application messages and schedule transmissions.
   - Include confirmed uplink acknowledgements as described in `techd.md`.

## Phase 2 – Extensions
4. **Integrations**
   - Publish events to external systems (databases, cloud services) through a configurable integration layer.
5. **gRPC / API Layer**
   - Expose management and data APIs similar to the ChirpStack API.
   - Keep protocol definitions separate from business logic.
6. **Monitoring and Metrics**
   - Emit metrics (e.g. Prometheus) and structured logs for observability.
7. **Additional Features**
   - Add multicast, firmware updates (FUOTA) and roaming interfaces.
   - Support multiple database backends (SQL or key-value) via repository abstractions.

Document the design and decisions as the implementation progresses to ensure the final server mirrors the features of this repository while staying technology independent.
