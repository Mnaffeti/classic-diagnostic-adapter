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

#### 5. DynamicallyDefineDataIdentifier (SID 0x2C)
- **Status**: Not implemented
- **ISO 14229 Reference**: Section 8.3.1.4
- **SOVD Mapping**: Could map to `/operations/define-data-identifier` or `/data/{data-identifier}` with dynamic creation
- **Use Case**: Dynamically define data identifiers at runtime
- **Implementation Requirements**:
  - Support subfunctions: DefineByIdentifier, DefineByMemoryAddress, ClearDynamicallyDefinedDataIdentifier
  - Handle dynamic data identifier lifecycle

#### 6. RequestUpload (SID 0x35)
- **Status**: Not implemented
- **ISO 14229 Reference**: Section 8.3.1.5
- **SOVD Mapping**: Could map to `/x-sovd2uds-upload/requestupload` (mirror of download)
- **Use Case**: Upload data from ECU (read ECU memory/flash)
- **Implementation Requirements**:
  - Similar to RequestDownload but for reading from ECU
  - Add upload endpoints: `/x-sovd2uds-upload/requestupload`, `/x-sovd2uds-upload/uploadtransfer`, `/x-sovd2uds-upload/transferexit`
  - Handle upload data transfer similar to download

### Lower Priority - Advanced/Extended Services

#### 7. AccessTimingParameter (SID 0x83)
- **Status**: Not implemented
- **ISO 14229 Reference**: Section 8.3.3.1
- **SOVD Mapping**: Could map to `/modes/timing` or `/operations/timing-parameter`
- **Use Case**: Set/get timing parameters for diagnostic communication
- **Implementation Requirements**:
  - Support subfunctions: SetTimingParametersToGivenValues, ReadCurrentlyActiveTimingParameters, SetTimingParametersToDefaultValues
  - Handle timing parameter configuration

#### 8. SecuredDataTransmission (SID 0x84)
- **Status**: Not implemented
- **ISO 14229 Reference**: Section 8.3.3.2
- **SOVD Mapping**: Could be handled transparently or via `/operations/secured-transmission`
- **Use Case**: Encrypted/secure data transmission
- **Implementation Requirements**:
  - Support secure data transmission protocols
  - Handle encryption/decryption
  - May require security plugin integration

#### 9. ResponseOnEvent (SID 0x86)
- **Status**: Not implemented
- **ISO 14229 Reference**: Section 8.3.3.3
- **SOVD Mapping**: Could map to `/operations/event-subscription` or `/modes/event-monitoring`
- **Use Case**: Subscribe to ECU events (DTCs, data changes, etc.)
- **Implementation Requirements**:
  - Support event subscription management
  - Handle asynchronous event notifications
  - Support multiple event types: OnDTCStatusChange, OnTimerInterrupt, OnChangeOfDataIdentifier, etc.
  - WebSocket or similar for real-time event delivery

#### 10. LinkControl (SID 0x87)
- **Status**: Not implemented
- **ISO 14229 Reference**: Section 8.3.3.4
- **SOVD Mapping**: Could map to `/modes/link-control` or `/operations/link-control`
- **Use Case**: Control communication link parameters
- **Implementation Requirements**:
  - Support subfunctions: VerifyBaudrateTransitionWithFixedBaudrate, VerifyBaudrateTransitionWithSpecificBaudrate, TransitionBaudrate
  - Handle baudrate transitions (mainly for CAN, less relevant for DoIP)

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
  
- **MIME Type Handling**:
  - Automatic MIME type detection
  - Content-Type headers in responses
  
- **Security Restrictions**:
  - Path-based access control (as per requirements: "source of data must be restrictable to a path and its subdirectories via configuration")
  - Prevent directory traversal attacks
  - File size limits
  - Allowed file type restrictions per category

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
- **Missing**: CAN, LIN, K-line, and other legacy transport protocols
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

### Implementation Architecture

The codebase is already designed to support multiple protocols:

1. **Gateway Trait** (`cda-interfaces/src/ecugateway.rs`):
   - `EcuGateway` trait abstracts transport layer
   - Handles protocol-specific ACKs/NACKs
   - Manages frame/message assembly
   - Already supports multiple protocols in design

2. **Protocol Enum** (`cda-interfaces/src/ecumanager.rs`):
   - Currently: `DoIp`, `DoIpDobt`
   - Needs: `Can`, `Lin`, `Kline` variants

3. **Communication Modules**:
   - Pattern: Create `cda-comm-{protocol}` crate
   - Example: `cda-comm-doip` can be used as reference
   - Each protocol implements `EcuGateway` trait

### Implementation Steps

1. **CAN Implementation** (Recommended First):
   ```
   cda-comm-can/
   ├── src/
   │   ├── lib.rs              # CAN gateway implementation
   │   ├── isotp.rs            # ISO-TP protocol handling
   │   ├── can_frame.rs        # CAN frame structure
   │   ├── can_socket.rs       # CAN socket abstraction
   │   └── connections.rs      # Connection management
   └── Cargo.toml
   ```

2. **Update Protocol Enum**:
   - Add `Can`, `Lin`, `Kline` to `Protocol` enum
   - Update protocol detection logic

3. **Update Communication Parameters**:
   - Add CAN-specific communication parameters
   - Add LIN-specific communication parameters
   - Add K-line-specific communication parameters
   - Reference: ISO 14229-3 for CAN, ISO 17987 for LIN

4. **ODX/MDD Support**:
   - Ensure MDD files can specify CAN/LIN/K-line logical links
   - Update ODX converter if needed

5. **Testing**:
   - CAN bus simulators (e.g., CANoe, Peak CAN)
   - Integration tests with CAN hardware
   - Test multi-frame messages (ISO-TP)

### Hardware Requirements

- **CAN**: 
  - SocketCAN (Linux) or vendor-specific CAN interfaces
  - Examples: Peak PCAN, Vector CAN interfaces, USB-CAN adapters
- **LIN**: 
  - LIN interfaces (often combined with CAN interfaces)
- **K-line**: 
  - K-line interfaces (OBD-II adapters, ELM327, etc.)

### Documentation Updates Needed

- Add CAN communication architecture docs
- Add LIN communication architecture docs  
- Add K-line communication architecture docs
- Update requirements documentation
- Add hardware setup guides

### Related Standards

- **ISO 14229-3**: UDS on CAN
- **ISO 15765-2**: ISO-TP (Transport Protocol)
- **ISO 11898**: CAN specification
- **ISO 17987**: LIN specification
- **ISO 9141-2**: K-line initialization
- **ISO 14230**: K-line protocol

## Recommended Contribution Priority

### Phase 1: Critical Missing Services
1. **CAN Protocol Support** - **HIGHEST PRIORITY** - Enables legacy vehicle support
2. **InputOutputControlByIdentifier (0x2A)** - Explicitly noted as missing
3. **Complete Faults Implementation** - Clear DTCs functionality
4. **Bulk Data Management** - Required by SOVD standard

### Phase 2: Important Standard Services
4. **ReadMemoryByAddress (0x23)** - Useful for debugging/calibration
5. **WriteMemoryByAddress (0x3D)** - Useful for calibration
6. **RequestUpload (0x35)** - Complement to download functionality

### Phase 3: Advanced Features
7. **ResponseOnEvent (0x86)** - Real-time event monitoring
8. **Functional Communication** - Group-based operations
9. **DynamicallyDefineDataIdentifier (0x2C)** - Dynamic data handling

### Phase 4: Extended Services
10. **AccessTimingParameter (0x83)** - Timing configuration
11. **SecuredDataTransmission (0x84)** - Security enhancements
12. **LinkControl (0x87)** - Link parameter control
13. **ReadScalingDataByIdentifier (0x24)** - Scaling information

### Phase 5: Parameter Type Support
14. **Complete Parameter Type Support** - All ODX parameter types

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

## References

### UDS Standards
- **ISO 14229-1**: Unified diagnostic services (UDS) - Specification and requirements
- **ISO 14229-2**: Unified diagnostic services (UDS) - Session layer services
- **ISO 14229-3**: Unified diagnostic services (UDS) - Implementation for CAN
- **ISO 14229-5**: Unified diagnostic services (UDS) - Implementation for DoIP

### Transport Protocol Standards
- **ISO 13400**: Diagnostics over IP (DoIP)
- **ISO 15765-2**: ISO-TP (Transport Protocol for CAN)
- **ISO 11898**: CAN (Controller Area Network) specification
- **ISO 17987**: LIN (Local Interconnect Network) specification
- **ISO 9141-2**: K-line initialization
- **ISO 14230**: K-line protocol (Keyword Protocol 2000)

### SOVD Standards
- **ISO 17978-1**: Service-Oriented Vehicle Diagnostics (SOVD) - Part 1: General information and use case definition
- **ISO 17978-3**: Service-Oriented Vehicle Diagnostics (SOVD) - Part 3: API specification

## Getting Started

1. Review the existing implementation patterns in the codebase
2. Check the architecture documentation for design decisions
3. Review similar service implementations (e.g., RoutineControl for Operations)
4. Create an issue or discussion in the GitHub repository
5. Follow the coding style guidelines (`CODESTYLE.md`)
6. Write tests alongside implementation
7. Update documentation

## Contact

For questions or to discuss contributions:
- GitHub Issues: https://github.com/eclipse-opensovd/classic-diagnostic-adapter/issues
- Mailing List: opensovd-dev@eclipse.org

