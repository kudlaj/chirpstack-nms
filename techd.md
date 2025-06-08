# Technical Design Overview

ChirpStack NMS (Network Management Server) is a LoRaWAN® Network Server
implementation written in Rust. The workspace contains multiple crates that
implement different layers of the system:

- **chirpstack**: main network server logic including gateway handling,
  uplink/downlink processing, database access and gRPC API setup.
- **backend**: client for the LoRaWAN Backend Interfaces used to communicate
  with Join Servers and roaming Network Servers.
- **chirpstack-integration**: reads events from Redis Streams and dispatches
  them to an integration implementation.
- **lrwn / lrwn-filters**: libraries for LoRaWAN protocol definitions and
  filtering helpers.
- **api**: protobuf definitions and generated gRPC code for external APIs.

## High level architecture
```mermaid
flowchart LR
    subgraph External
        App[Application]
        JS[Join Server]
        Roaming[Other NS]
    end
    Device --RF--> Gateway
    Gateway --MQTT--> NS(ChirpStack Network Server)
    NS --Backend Interfaces--> JS
    NS --Backend Interfaces--> Roaming
    NS --"Redis Streams"--> Integration
    Integration --callbacks--> App
    NS --gRPC--> APIClients
```

## Uplink processing
The MQTT gateway backend collects uplink frames and performs deduplication before
handling them. A simplified flow is shown below.

```mermaid
flowchart TD
    GW[Gateway] --uplink--> NS
    NS --> Dedup["Deduplicate uplinks"]
    Dedup --> Proc["Process uplink"]
    Proc -->|Join requests| JS
    Proc -->|Data events| Integration
```

### Deduplication flow
When multiple gateways forward the same uplink, ChirpStack merges them before
processing. The events are stored in Redis and a short lock prevents multiple
workers from handling the same frame set.

```mermaid
flowchart TD
    A[Receive uplink] --> B[Store in Redis set]
    B --> C{Lock obtained?}
    C -- No --> G[Return]
    C -- Yes --> D[Wait deduplication delay]
    D --> E[Collect all frames]
    E --> F[Process UplinkFrameSet]
```

## Downlink processing
External applications enqueue downlinks through the gRPC API. The network server
schedules the frame and forwards it to the gateway.

```mermaid
flowchart TD
    App --gRPC--> NS
    NS -->|Schedule| GW
    GW --RF--> Device
```

## Confirmed uplink flow
When a device sends a confirmed uplink, the network server records that an ACK
must be sent. On the next downlink window (RX1 or RX2) it transmits a frame with
the `ACK` bit set in the FCtrl field. If an application downlink is already
queued, the ACK is piggy‑backed on that frame. Otherwise the server schedules an
empty downlink just to convey the acknowledgement. After the transmission the
downlink counter is updated and the pending ACK flag is cleared.

```mermaid
flowchart TD
    Device --confirmed uplink--> GW
    GW --> NS
    NS -->|ACK| GW
    GW --RF--> Device
```

The server also exposes metrics through Prometheus and logs using the tracing
crate. Redis and a SQL database (PostgreSQL or SQLite) are used for runtime
state and persistent storage respectively.
