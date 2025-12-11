# Contribution Opportunities for Eclipse OpenSOVD Classic Diagnostic Adapter

## Overview

This document identifies missing UDS services and SOVD endpoints that need to be implemented to improve compliance with ISO 14229 (UDS) and ISO 17978 (SOVD) standards.

## Current Implementation Status

### Implemented UDS Services (Mapped to SOVD)

| UDS SID | Service Name | SOVD Endpoint | Status |
|---------|--------------|---------------|--------|
| 0x10 | DiagnosticSessionControl | `/modes/session` | ✅ Implemented |
| 0x11 | ECUReset | `/operations/reset`, `/status/restart` | ✅ Implemented |
| 0x14 | ClearDiagnosticInformation | `/faults/` (partial) | ⚠️ Partially Implemented |
| 0x19 | ReadDTCInformation | `/faults/` | ✅ Implemented |
| 0x22 | ReadDataByIdentifier | `/data/{data-identifier}` | ✅ Implemented |
| 0x27 | SecurityAccess | `/modes/security` | ✅ Implemented |
| 0x28 | CommunicationControl | `/modes/commctrl` | ✅ Implemented |
| 0x29 | Authentication | `/modes/authentication` | ✅ Implemented |
| 0x2E | WriteDataByIdentifier | `/configurations/{data-identifier}` | ✅ Implemented |
| 0x31 | RoutineControl | `/operations/{routine-identifier}` | ✅ Implemented |
| 0x34 | RequestDownload | `/x-sovd2uds-download/requestdownload` | ✅ Implemented |
| 0x36 | TransferData | `/x-sovd2uds-download/flashtransfer` | ✅ Implemented |
| 0x37 | RequestTransferExit | `/x-sovd2uds-download/transferexit` | ✅ Implemented |
| 0x3E | TesterPresent | Internal (handled automatically) | ✅ Implemented |
| 0x85 | ControlDTCSetting | `/modes/dtcsetting` | ✅ Implemented |

## Missing UDS Services to SOVD Endpoints

### High Priority - Explicitly Noted as Missing

#### 1. InputOutputControlByIdentifier (SID 0x2A)
- **Status**: Explicitly marked as "Not supported at this time" in architecture docs
- **Location**: `docs/architecture/03_sovd-api/02_sovd-api.adoc:323`
- **ISO 14229 Reference**: Section 8.3.2.2
- **SOVD Mapping**: Should map to `/operations/` or a dedicated `/io-control/` endpoint
- **Use Case**: Control input/output signals on ECUs (e.g., activate actuators, control relays)
- **Implementation Requirements**:
  - Add SOVD endpoint for IOControl operations
  - Support subfunctions: ReturnControlToECU, ResetToDefault, FreezeCurrentState, ShortTermAdjustment
  - Map to UDS SID 0x2A with proper parameter handling
  - Add to `OPERATIONS_PREFIXES` in `cda-interfaces/src/lib.rs`

### Medium Priority - Standard UDS Services Not Mapped

#### 2. ReadMemoryByAddress (SID 0x23)
- **Status**: Not implemented
- **ISO 14229 Reference**: Section 8.3.1.2
- **SOVD Mapping**: Could map to `/data/{data-identifier}` with special handling or `/operations/read-memory`
- **Use Case**: Read ECU memory by address (useful for debugging, calibration)
- **Implementation Requirements**:
  - Add support for memory address-based reads
  - Handle address and length parameters
  - Consider if this should be part of data resources or operations

#### 3. WriteMemoryByAddress (SID 0x3D)
- **Status**: Not implemented
- **ISO 14229 Reference**: Section 8.3.2.1
- **SOVD Mapping**: Could map to `/configurations/{data-identifier}` or `/operations/write-memory`
- **Use Case**: Write ECU memory by address (calibration, debugging)
- **Implementation Requirements**:
  - Add support for memory address-based writes
  - Handle address, data, and length parameters
  - Security considerations (may require elevated security access)

#### 4. ReadScalingDataByIdentifier (SID 0x24)
- **Status**: Not implemented
- **ISO 14229 Reference**: Section 8.3.1.3
- **SOVD Mapping**: Could map to `/data/{data-identifier}` with scaling metadata
- **Use Case**: Read scaling information for data identifiers
- **Implementation Requirements**:
  - Add support for scaling data reads
  - Return scaling information in response


#### 5. RequestUpload (SID 0x35)
- **Status**: Not implemented
- **ISO 14229 Reference**: Section 8.3.1.5
- **SOVD Mapping**: Could map to `/x-sovd2uds-upload/requestupload` (mirror of download)
- **Use Case**: Upload data from ECU (read ECU memory/flash)
- **Implementation Requirements**:
  - Similar to RequestDownload but for reading from ECU
  - Add upload endpoints: `/x-sovd2uds-upload/requestupload`, `/x-sovd2uds-upload/uploadtransfer`, `/x-sovd2uds-upload/transferexit`
  - Handle upload data transfer similar to download


## Incomplete Implementations

### 1. Faults Endpoint (SID 0x14 & 0x19)
- **Status**: Partially implemented, marked as `#TODO#` in architecture docs
- **Location**: `docs/architecture/03_sovd-api/02_sovd-api.adoc:415`
- **Current State**: 
  - GET `/faults` is implemented (reads DTCs)
  - GET `/faults/{id}` is implemented (extended fault info)
  - Missing: Clear DTCs functionality (SID 0x14)
- **Implementation Requirements**:
  - Add DELETE `/faults/{id}` or POST `/faults/{id}/clear` to clear individual DTCs
  - Add DELETE `/faults` or POST `/faults/clear` to clear all DTCs
  - Support different clear types: ClearDTCs, ClearDTCsAndStoredData, ClearDTCsAndStoredDataAndResetStatus

### 2. Parameter Type Support
- **Status**: Several parameter types not fully supported
- **Location**: `cda-core/src/diag_kernel/ecumanager.rs`
- **Missing Types**:
  - `TableStruct` - Not implemented for request mapping
  - `Dynamic` - Not implemented
  - `LengthKey` - Not implemented
  - `NrcConst` - Not implemented
  - `PhysConst` - Not implemented
  - `System` - Not implemented
  - `TableEntry` - Not implemented
  - `TableKey` - Not implemented
- **Implementation Requirements**:
  - Implement mapping for all parameter types in both directions (to/from UDS)
  - Add proper error handling for unsupported types
  - Update documentation

## Missing SOVD Features (ISO 17978 Compliance)

### 1. Bulk Data Management
- **Status**: Partially implemented (read-only operations exist, write operations missing)
- **Location**: `docs/architecture/03_sovd-api/02_sovd-api.adoc:115-152`
- **Purpose**: Bulk data endpoints manage large files/data used for operations like ECU flashing, calibration files, or other binary data that needs to be stored and retrieved separately from regular diagnostic operations.

#### What Bulk Data Does:
- **List Bulk Data (GET `/bulk-data/{category}`)**: 
  - Returns a list of all files/entries in a specific category
  - Each entry includes: `id`, `mimetype`, `size`, `hash` (optional), `origin_path`
  - Example categories: `flashfiles`, `mdd-embedded-files`, `calibration-files`
  - **Current Implementation**: ✅ Partially implemented
    - Flash files: `GET /apps/sovd2uds/bulk-data/flashfiles` - lists files from configured flash directory
    - MDD embedded files: `GET /components/{ecu}/x-sovd2uds-bulk-data/mdd-embedded-files` - lists files embedded in MDD

- **Upload Bulk Data (POST `/bulk-data/{category}`)**:
  - Uploads a file to a specific category for later use
  - **Use Case**: Upload flash files before flashing an ECU, upload calibration files, etc.
  - Metadata (e.g., filename) can be provided via `Content-Disposition: form-data`
  - **Current Implementation**: ❌ **NOT IMPLEMENTED** - This is a critical missing feature
  - **Implementation Requirements**:
    - Accept file uploads via multipart/form-data or binary stream
    - Store files in category-specific directories
    - Generate unique entry IDs
    - Validate file types and sizes
    - Apply security restrictions (path-based access control per requirements)

- **Download Bulk Data (GET `/bulk-data/{category}/{entry-id}`)**:
  - Downloads a specific file from a category
  - MIME type is determined by the server based on file content
  - **Current Implementation**: ✅ Partially implemented
    - MDD embedded files: `GET /components/{ecu}/x-sovd2uds-bulk-data/mdd-embedded-files/{id}` - downloads embedded files
    - Flash files download: ❌ Not implemented (files are read directly from filesystem)

- **Delete Bulk Data (DELETE `/bulk-data/{category}` or `/bulk-data/{category}/{entry-id}`)**:
  - Removes files from categories
  - Can delete all entries in a category or a specific entry
  - **Current Implementation**: ❌ **NOT IMPLEMENTED**

#### Implementation Requirements:
- **File Storage Management**: 
  - Persistent storage for uploaded files
  - Category-based directory structure
  - File metadata tracking (size, MIME type, hash, upload timestamp)
  
- **Category-Based Organization**:
  - Support multiple categories (e.g., `flashfiles`, `calibration`, `logs`)
  - Category-specific access controls
  

#### Example Use Case - ECU Flashing:
1. **Upload**: `POST /bulk-data/flashfiles` - Upload ECU firmware file
2. **List**: `GET /bulk-data/flashfiles` - Verify file is available
3. **Flash**: Use the file ID in flash transfer operations (`/x-sovd2uds-download/flashtransfer`)
4. **Delete**: `DELETE /bulk-data/flashfiles/{file-id}` - Clean up after flashing

### 2. Functional Communication
- **Status**: Architecture defined, implementation status unclear
- **Location**: `docs/requirements/04_sovd.adoc:250-278`
- **Required Endpoints**:
  - `/functions/functionalgroups/{groupName}/locks`
  - `/functions/functionalgroups/{groupName}/operations`
  - `/functions/functionalgroups/{groupName}/data`
  - `/functions/functionalgroups/{groupName}/modes`
- **Implementation Requirements**:
  - Functional addressing support
  - Group-based ECU management
  - Functional tester present handling

### 3. Vehicle-Level Operations
- **Status**: Architecture defined
- **Location**: `docs/requirements/04_sovd.adoc:230-248`
- **Required Resources**:
  - `/locks` - Vehicle-level locks
  - `/functions` - Vehicle-level functional communication
- **Implementation Requirements**:
  - Vehicle-wide lock management
  - MDD file updates at runtime
  - Reserve all ECUs functionality

## Legacy Communication Protocol Support

### Current Status
- **Supported**: Only DoIP (Diagnostics over IP - Ethernet-based, ISO 13400)
- **Missing**: CAN, LIN, K-line, J2534 PassThru, and other legacy transport protocols
- **Priority**: **HIGH** - Critical for supporting legacy vehicles and ECUs

### Evidence in Codebase
- **Location**: `cda-interfaces/src/ecumanager.rs:50` - Comment: `// todo: other protocols`
- **Location**: `cda-interfaces/src/ecumanager.rs:64` - Comment mentions CAN: "It might be the case, that no all functions are needed for every protocol. (I.e. gateway address for CAN)."
- **Protocol Enum**: Currently only has `DoIp` and `DoIpDobt` variants
- **Architecture**: The `EcuGateway` trait is designed to support multiple protocols, but only DoIP implementation exists

### Why This Matters
1. **Legacy Vehicle Support**: Many vehicles still use CAN-based diagnostics
2. **Market Coverage**: Enables CDA to work with older vehicles and ECUs
3. **Gateway Scenarios**: Many modern vehicles have CAN ECUs behind DoIP gateways
4. **Industry Standard**: ISO 14229-3 defines UDS over CAN, which is widely used

### Required Implementations

#### 1. CAN (Controller Area Network) - ISO 14229-3
- **Status**: ❌ Not implemented
- **Standard**: ISO 14229-3 (UDS on CAN), ISO-TP (ISO 15765-2)
- **Priority**: **HIGHEST** - Most common legacy protocol
- **Implementation Requirements**:
  - ISO-TP (ISO Transport Protocol) for multi-frame message handling
  - CAN frame assembly/disassembly
  - Flow control handling (FC - Flow Control frames)
  - CAN addressing (11-bit and 29-bit identifiers)
  - Physical vs Functional addressing
  - Gateway address handling (for ECUs behind gateways)
  - Timing parameters (N_As, N_Ar, N_Bs, N_Br, N_Cs, N_Cr)
  - STmin and BS (Block Size) handling
  - Create new crate: `cda-comm-can` (similar to `cda-comm-doip`)
  - Implement `EcuGateway` trait for CAN
  - Add CAN protocol variant to `Protocol` enum
  - CAN hardware abstraction layer (socketCAN on Linux, or vendor-specific APIs)

#### 2. LIN (Local Interconnect Network)
- **Status**: ❌ Not implemented
- **Standard**: ISO 14229-3 (UDS on LIN), ISO 17987
- **Priority**: **MEDIUM** - Less common, mainly for body electronics
- **Implementation Requirements**:
  - LIN frame handling
  - UDS over LIN protocol implementation
  - Create `cda-comm-lin` crate
  - Implement `EcuGateway` trait for LIN

#### 3. K-line (ISO 9141-2 / ISO 14230)
- **Status**: ❌ Not implemented
- **Standard**: ISO 14229-3 (UDS on K-line), ISO 9141-2, ISO 14230
- **Priority**: **MEDIUM** - Older vehicles, mainly European
- **Implementation Requirements**:
  - K-line initialization (5-baud vs fast init)
  - K-line frame handling
  - UDS over K-line protocol
  - Create `cda-comm-kline` crate
  - Implement `EcuGateway` trait for K-line
  - Hardware abstraction for K-line interfaces

#### 4. J2534 PassThru API - SAE J2534
- **Status**: ❌ Not implemented
- **Standard**: SAE J2534-1, SAE J2534-2
- **Priority**: **HIGH** - Industry-standard API for aftermarket diagnostic tools
- **Why It Matters**:
  - **Multi-Protocol Support**: Single API supports CAN, ISO15765, ISO14230, ISO9141, J1850, and more
  - **Standardized Interface**: SAE standard ensures compatibility across vendors
  - **No Hardware Lock-in**: Users can choose their preferred PassThru device
  - **Cost-Effective**: Leverages existing commercial hardware investments

**Implementation Requirements**:

**A. Core J2534 Gateway (`cda-comm-j2534` crate)**:
```
cda-comm-j2534/
├── Cargo.toml
└── src/
    ├── lib.rs                  # J2534DiagGateway implementing EcuGateway trait
    ├── connections.rs          # Connection pool and device management
    ├── device_connection.rs    # PassThru device connection handling
    ├── channel_manager.rs      # J2534 channel lifecycle management
    └── protocol_translator.rs  # UDS-to-J2534 protocol mapping
```

**B. Architecture Components**:

1. **J2534DiagGateway Structure** (similar to `DoipDiagGateway`):
   - Implements `EcuGateway` trait from `cda-interfaces`
   - Manages multiple J2534 devices (connection pool)
   - Maps ECU logical addresses to J2534 channels
   - Handles asynchronous read/write operations
   - Manages tester present via periodic messaging

2. **J2534Device Connection Manager**:
   - Device enumeration (PassThruOpen)
   - Channel management (PassThruConnect/PassThruDisconnect)
   - Filter configuration (PassThruStartMsgFilter)
   - Message queuing (async TX/RX queues)
   - Error handling and retry logic

3. **Protocol Translators**:
   - ISO15765 (CAN/ISO-TP) translator
   - ISO14230 (KWP2000) translator
   - ISO9141 (K-Line) translator
   - J1850 VPW/PWM translators
   - Protocol-specific frame assembly/disassembly

4. **FFI Bindings**:
   - Use existing Rust J2534 crate (e.g., `j2534` or `j2534-rs`)
   - OR create custom FFI bindings to J2534 DLLs
   - Dynamic library loading (libloading)

**C. Interface Extensions** (`cda-interfaces/src/com_param_handling.rs`):

Add new trait `J2534ComParamProvider`:
```rust
pub trait J2534ComParamProvider: Send + Sync + 'static {
    fn device_name(&self) -> String;              // PassThru device name
    fn protocol(&self) -> J2534Protocol;          // CAN, ISO15765, ISO14230, etc.
    fn baudrate(&self) -> u32;                    // Bus baudrate
    fn connection_flags(&self) -> u32;            // Protocol-specific flags
    fn filters(&self) -> Vec<FilterConfig>;       // Message filters
    fn timeout_read(&self) -> Duration;           // Read timeout
    fn timeout_write(&self) -> Duration;          // Write timeout
    fn can_extended_addressing(&self) -> bool;    // Extended vs standard addressing
    fn can_id_tx(&self) -> Option<u32>;          // CAN TX ID (if applicable)
    fn can_id_rx(&self) -> Option<u32>;          // CAN RX ID (if applicable)
}

pub enum J2534Protocol {
    J1850VPW,
    J1850PWM,
    ISO9141,
    ISO14230,     // KWP2000
    CAN,
    ISO15765,     // CAN with ISO-TP
    // Additional protocols...
}
```

**D. Configuration Extension** (`opensovd-cda.toml`):
```toml
[communication]
# Existing
doip_enabled = true
tester_address = "192.168.1.100"

# NEW J2534 Configuration
j2534_enabled = true
j2534_device = "PassThruDevice.04.04"  # Device name from Windows registry
j2534_protocol = "ISO15765"            # Default protocol
j2534_can_baudrate = 500000

```

**E. MDD Database Extensions** (`cda-database`):

Extend MDD parser to support J2534 communication parameters:
```rust
pub struct J2534ComParams {
    pub device_name: String,
    pub protocol: J2534Protocol,
    pub baudrate: u32,
    pub can_id_tx: Option<u32>,       // CAN TX identifier
    pub can_id_rx: Option<u32>,       // CAN RX identifier
    pub connection_flags: u32,
    pub filters: Vec<FilterConfig>,
    pub timeout_read: Duration,
    pub timeout_write: Duration,
}
```

**F. Implementation Pattern** (following DoIP pattern):

1. **Connection Establishment**:
   ```
   1. Enumerate available PassThru devices (PassThruOpen)
   2. Select device based on MDD configuration
   3. Open logical channel with protocol (PassThruConnect)
   4. Configure message filters (PassThruStartMsgFilter)
   5. Start async read/write tasks (Tokio tasks)
   6. Map ECU logical addresses to channels
   7. Handle tester present via periodic messaging
   ```

2. **Message Flow** (similar to `DoipDiagGateway::send`):
   ```rust
   async fn send(
       &self,
       transmission_params: TransmissionParameters,
       message: ServicePayload,
       response_sender: mpsc::Sender<Result<Option<UdsResponse>, DiagServiceError>>,
       expect_uds_reply: bool,
   ) -> Result<(), DiagServiceError> {
       // 1. Get J2534 device and channel
       // 2. Convert UDS ServicePayload to PassThruMsg
       // 3. Send via PassThruWriteMsgs
       // 4. Wait for responses via async receive task
       // 5. Convert PassThruMsg back to UdsResponse
       // 6. Handle retries and timeouts
   }
   ```

3. **Connection Handler** (similar to `handle_gateway_connection`):
   ```rust
   async fn handle_device_connection<T>(
       device: J2534Target,
       j2534_devices: &Arc<RwLock<Vec<Arc<J2534Device>>>>,
       ecus: &Arc<HashMap<String, RwLock<T>>>,
       ecu_channel_map: &HashMap<u16, Vec<u16>>,
   ) -> Result<u16, J2534Error>
   where
       T: EcuAddressProvider + J2534ComParamProvider
   ```

**G. Dependencies** (`Cargo.toml`):
```toml
[workspace.dependencies]
# J2534 FFI bindings
j2534 = "0.3"              # OR j2534-rs
libloading = "0.8"         # Dynamic library loading
# Existing dependencies...
```

### Implementation Architecture

The codebase is already designed to support multiple protocols:

1. **Gateway Trait** (`cda-interfaces/src/ecugateway.rs`):
   - `EcuGateway` trait abstracts transport layer
   - Handles protocol-specific ACKs/NACKs
   - Manages frame/message assembly
   - Already supports multiple protocols in design

2. **Protocol Enum** (`cda-interfaces/src/ecumanager.rs`):
   - Currently: `DoIp`, `DoIpDobt`
   - Needs: `Can`, `Lin`, `Kline`, `J2534` variants

3. **Communication Modules**:
   - Pattern: Create `cda-comm-{protocol}` crate
   - Example: `cda-comm-doip` can be used as reference
   - Each protocol implements `EcuGateway` trait
   - J2534 follows same pattern: `cda-comm-j2534`

### Implementation Steps

1. **Priority 1: J2534 PassThru API** (Recommended First for maximum hardware compatibility):
   ```
   cda-comm-j2534/
   ├── src/
   │   ├── lib.rs              # J2534DiagGateway implementation
   │   ├── device_connection.rs # PassThru device handling
   │   ├── channel_manager.rs  # J2534 channel management
   │   ├── protocol_translator.rs # UDS-to-J2534 translation
   │   └── connections.rs      # Connection pool management
   └── Cargo.toml
   ```
   **Benefits**: Single implementation supports CAN, K-Line, and other protocols via standard API

2. **Priority 2: Native CAN Implementation** (For direct CAN hardware):
   ```
   cda-comm-can/
   ├── src/
   │   ├── lib.rs              # CAN gateway implementation
   │   ├── isotp.rs            # ISO-TP protocol handling
   │   ├── can_frame.rs        # CAN frame structure
   │   ├── can_socket.rs       # CAN socket abstraction (socketCAN)
   │   └── connections.rs      # Connection management
   └── Cargo.toml
   ```

3. **Update Protocol Enum**:
   - Add `Can`, `Lin`, `Kline`, `J2534` to `Protocol` enum
   - Update protocol detection logic

4. **Update Communication Parameters**:
   - Add J2534-specific communication parameters (`J2534ComParamProvider`)
   - Add CAN-specific communication parameters
   - Add LIN-specific communication parameters
   - Add K-line-specific communication parameters
   - Reference: ISO 14229-3 for CAN, ISO 17987 for LIN, SAE J2534 for PassThru

5. **ODX/MDD Support**:
   - Ensure MDD files can specify J2534/CAN/LIN/K-line logical links
   - Update ODX converter if needed


## Recommended Contribution Priority

### Phase 1: Critical Missing Services
1. **J2534 PassThru API Support** - **HIGHEST PRIORITY** - Universal hardware interface for all protocols (CAN, K-Line, etc.)
2. **CAN Protocol Support** - **HIGH PRIORITY** - Direct CAN support for native hardware (alternative/complement to J2534)
3. **InputOutputControlByIdentifier (0x2A)** - Explicitly noted as missing
4. **Complete Faults Implementation** - Clear DTCs functionality
5. **Bulk Data Management** - Required by SOVD standard

### Phase 2: Important Standard Services
6. **ReadMemoryByAddress (0x23)** - Useful for debugging/calibration
7. **WriteMemoryByAddress (0x3D)** - Useful for calibration
8. **RequestUpload (0x35)** - Complement to download functionality
9. **LIN Protocol Support** - Body electronics and lower-cost ECUs
10. **K-Line Protocol Support** - Legacy vehicle support

### Phase 3: Advanced Features
11. **ResponseOnEvent (0x86)** - Real-time event monitoring
12. **Functional Communication** - Group-based operations
13. **DynamicallyDefineDataIdentifier (0x2C)** - Dynamic data handling

### Phase 4: Extended Services
14. **AccessTimingParameter (0x83)** - Timing configuration
15. **SecuredDataTransmission (0x84)** - Security enhancements
16. **LinkControl (0x87)** - Link parameter control
17. **ReadScalingDataByIdentifier (0x24)** - Scaling information

### Phase 5: Parameter Type Support
18. **Complete Parameter Type Support** - All ODX parameter types

## Implementation Guidelines

### For Each Service Implementation:

1. **UDS Layer** (`cda-comm-uds`):
   - Add service ID constant to `cda-interfaces/src/lib.rs`
   - Implement UDS request/response handling
   - Add to appropriate prefix arrays (DATA_PREFIXES, OPERATIONS_PREFIXES, etc.)

2. **Core Layer** (`cda-core`):
   - Add service lookup and payload conversion
   - Handle parameter mapping (to/from UDS)
   - Support variant detection if needed

3. **SOVD Layer** (`cda-sovd`):
   - Create REST endpoint handlers
   - Add OpenAPI documentation
   - Implement request/response mapping
   - Add error handling

4. **Interfaces** (`cda-interfaces`):
   - Add trait methods if needed
   - Update data types
   - Add to service ID constants

5. **Documentation**:
   - Update architecture docs (`docs/architecture/`)
   - Update requirements docs (`docs/requirements/`)
   - Add examples and use cases

6. **Testing**:
   - Unit tests for UDS layer
   - Integration tests for SOVD endpoints
   - Test with ECU simulator

