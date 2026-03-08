# Study 004: pypykatz Total Architecture and Feature Analysis

## Executive Summary

This study provides a comprehensive analysis of pypykatz's overall architecture, complete function and feature mapping, and development roadmap. It builds upon the top 10 password acquisition strategies to present the complete pypykatz ecosystem.

## pypykatz Architecture Overview

```
pypykatz (Root)
├── Core Engine
│   ├── Main Entry Point (__main__.py)
│   ├── Command Line Interface (argparse)
│   └── Universal Encoder/Decoder
├── Memory Access Layer
│   ├── Live Readers (Windows APIs)
│   ├── Offline Readers (File parsers)
│   └── Remote Readers (Network protocols)
├── Credential Extraction Engines
│   ├── LSASS Decryptor
│   ├── Registry Parser
│   ├── DPAPI Handler
│   └── Kerberos Processor
├── Authentication Package Handlers
│   ├── MSV (NTLM)
│   ├── WDigest
│   ├── Kerberos
│   ├── SSP/LiveSSP
│   ├── TsPkg
│   ├── CredMan
│   └── CloudAP
├── Utility Modules
│   ├── Cryptographic Functions
│   ├── Network Operations
│   └── File Parsers
└── Output Formatters
    ├── JSON Export
    ├── Text Output
    └── Grep Format
```

## Hierarchical Architecture Analysis

### Level 1: Core Framework
```
pypykatz/
├── __main__.py          # Entry point and command dispatcher
├── pypykatz.py          # Main pypykatz class and orchestration
├── _version.py          # Version and banner information
└── commons/             # Shared utilities and data structures
    ├── common.py        # Universal encoder, system info
    ├── filetime.py      # Windows timestamp conversion
    ├── kerberosticket.py # Kerberos ticket handling
    └── win_datatypes.py # Windows data structure definitions
```

### Level 2: Data Access Layer
```
commons/readers/         # Abstracted data access layer
├── local/              # Live system access (Windows only)
│   ├── live_reader.py  # Direct memory access via Windows APIs
│   └── common/         # Local system utilities
├── registry/           # Registry hive parsing
│   ├── live_parser.py  # Live registry access
│   └── offline_parser.py # Registry hive file parsing
├── rekall/             # Memory dump analysis via Rekall
│   └── rekallreader.py # Rekall framework integration
├── volatility3/        # Volatility 3 integration
│   └── volreader.py    # Vol3 framework integration
└── zipdump/            # Compressed dump analysis
    └── reader.py       # ZIP-based dump processing
```

### Level 3: Credential Extraction Engines
```
lsadecryptor/           # LSASS memory analysis engine
├── lsa_decryptor.py    # Main LSA decryption orchestrator
├── lsa_decryptor_nt5.py # Windows NT5 (XP/2003) support
├── lsa_decryptor_nt6.py # Windows NT6+ (Vista/7/8/10/11) support
├── lsa_templates.py    # Memory structure templates
└── packages/           # Authentication package handlers
    ├── msv/            # MSV (NTLM) credential extraction
    ├── wdigest/        # WDigest plaintext passwords
    ├── kerberos/       # Kerberos tickets and keys
    ├── ssp/            # Security Support Provider
    ├── livessp/        # Live SSP credentials
    ├── tspkg/          # Terminal Services Package
    ├── credman/        # Windows Credential Manager
    ├── dpapi/          # DPAPI masterkeys
    └── cloudap/        # Azure AD/CloudAP credentials
```

### Level 4: Specialized Modules
```
registry/               # Windows Registry analysis
├── sam/                # SAM hive (local user accounts)
├── security/           # SECURITY hive (LSA secrets)
├── software/           # SOFTWARE hive (application data)
└── system/             # SYSTEM hive (boot keys)

dpapi/                  # Data Protection API
├── dpapi.py            # Main DPAPI orchestrator
├── structures/         # DPAPI data structures
│   ├── blob.py         # DPAPI blob parsing
│   ├── masterkeyfile.py # Master key file handling
│   ├── credentialfile.py # Credential file parsing
│   └── vault.py        # Windows Vault parsing
└── finders/            # DPAPI artifact discovery
    ├── cryptokeys.py   # Cryptographic key discovery
    ├── ngc.py          # Windows Hello NGC keys
    └── registry.py     # Registry-based DPAPI keys

kerberos/               # Kerberos ticket processing
├── kerberos.py         # Offline Kerberos analysis
├── kerberoslive.py     # Live Kerberos extraction
├── kirbiutils.py       # Kirbi ticket utilities
└── functiondefs/       # Kerberos API definitions
    ├── advapi32.py     # Windows Advanced API
    ├── kernel32.py     # Windows Kernel API
    └── netsecapi.py    # Network Security API
```

### Level 5: Network and Remote Operations
```
smb/                    # SMB-based remote operations
├── dcsync.py           # DCSync attack implementation
├── lsassutils.py       # Remote LSASS utilities
├── regutils.py         # Remote registry access
├── shareenum.py        # SMB share enumeration
└── printer.py          # SMB printer exploitation

remote/                 # Remote system access
├── live/               # Live remote operations
│   ├── common/         # Remote common utilities
│   ├── localgroup/     # Local group enumeration
│   ├── session/        # Session management
│   └── share/          # Share access utilities
└── cmdhelper.py        # Remote command helpers

ldap/                   # LDAP operations
├── cmdhelper.py        # LDAP command interface
└── (LDAP functionality) # Active Directory queries
```

## Complete Function and Feature Matrix

### Core Capabilities

| Category | Function | Implementation | Status | Complexity |
|----------|----------|----------------|--------|------------|
| **Memory Access** | Live LSASS Reading | Windows APIs | ✅ Stable | Medium |
| | Minidump Parsing | minidump library | ✅ Stable | Low |
| | Memory Dump Analysis | Rekall/Volatility | ✅ Stable | High |
| | Remote Memory Access | SMB/RPC | ✅ Stable | High |
| **Registry Analysis** | Live Registry | Windows APIs | ✅ Stable | Low |
| | Offline Hive Parsing | Binary parsing | ✅ Stable | Medium |
| | Remote Registry | SMB/RPC | ✅ Stable | Medium |
| **Credential Extraction** | NTLM Hashes | MSV package | ✅ Stable | Low |
| | Plaintext Passwords | WDigest package | ✅ Stable | Low |
| | Kerberos Tickets | Kerberos package | ✅ Stable | Medium |
| | DPAPI Keys | DPAPI package | ✅ Stable | High |
| | Cached Credentials | Registry parsing | ✅ Stable | Medium |

### Authentication Package Support

| Package | Purpose | Data Extracted | Windows Versions | Implementation Status |
|---------|---------|-----------------|------------------|---------------------|
| **MSV** | NTLM Authentication | NT/LM hashes, usernames | All | ✅ Complete |
| **WDigest** | HTTP Digest Auth | Plaintext passwords | Vista+ | ✅ Complete |
| **Kerberos** | Kerberos Authentication | Tickets, keys, passwords | All | ✅ Complete |
| **SSP** | Security Support Provider | Plaintext passwords | All | ✅ Complete |
| **LiveSSP** | Live SSP | Plaintext passwords | Win8+ | ✅ Complete |
| **TsPkg** | Terminal Services | Plaintext passwords | All | ✅ Complete |
| **CredMan** | Credential Manager | Stored credentials | Vista+ | ✅ Complete |
| **DPAPI** | Data Protection API | Master keys, blobs | All | ✅ Complete |
| **CloudAP** | Azure AD/Cloud | Cloud credentials | Win10+ | ✅ Complete |

### Data Source Compatibility

| Data Source | Format | Access Method | Cross-Platform | Performance |
|-------------|--------|---------------|----------------|-------------|
| **Live LSASS** | Process Memory | Windows APIs | ❌ Windows Only | ⚡ Fast |
| **LSASS Minidump** | .dmp file | File parsing | ✅ All Platforms | ⚡ Fast |
| **Full Memory Dump** | .vmem, .raw | Memory analysis | ✅ All Platforms | 🐌 Slow |
| **Registry Hives** | Binary files | File parsing | ✅ All Platforms | ⚡ Fast |
| **Hibernation File** | hiberfil.sys | Compressed memory | ✅ All Platforms | 🔄 Medium |
| **Crash Dumps** | .dmp file | File parsing | ✅ All Platforms | 🔄 Medium |
| **Remote Systems** | Network | SMB/RPC | ❌ Windows Target | 🔄 Medium |

### Output Format Support

| Format | Use Case | Structure | Automation Friendly |
|--------|----------|-----------|-------------------|
| **Text** | Human reading | Hierarchical display | ❌ No |
| **JSON** | Programmatic use | Structured data | ✅ Yes |
| **Grep** | Command-line filtering | Colon-separated | ✅ Yes |
| **CCACHE** | Kerberos tickets | MIT Kerberos format | ✅ Yes |
| **Kirbi** | Kerberos tickets | Mimikatz format | ✅ Yes |

## Advanced Feature Analysis

### Cryptographic Capabilities

```
utils/crypto/           # Cryptographic utilities
├── cmdhelper.py        # Crypto command interface
├── gppassword.py       # Group Policy password decryption
├── ofcdecrypt.py       # Office document decryption
├── others.py           # Miscellaneous crypto functions
└── winhash.py          # Windows hash calculations
```

| Function | Algorithm | Purpose | Implementation |
|----------|-----------|---------|----------------|
| **LSA Decryption** | AES-256, 3DES | LSASS credential decryption | unicrypto library |
| **Registry Decryption** | RC4 | Registry hive decryption | Custom implementation |
| **DPAPI Decryption** | AES-256 | DPAPI blob decryption | unicrypto library |
| **GP Password** | AES-256 | Group Policy password decrypt | Custom implementation |
| **Office Decrypt** | Various | Office document decryption | Custom implementation |
| **Hash Calculation** | MD4, SHA1 | Windows hash computation | hashlib |

### Network Protocol Support

| Protocol | Purpose | Implementation | Authentication |
|----------|---------|----------------|----------------|
| **SMB** | File sharing, remote access | aiosmb library | NTLM, Kerberos |
| **RPC** | Remote procedure calls | Custom implementation | NTLM, Kerberos |
| **LDAP** | Directory services | msldap library | NTLM, Kerberos |
| **MS-DRSR** | Directory replication (DCSync) | Custom implementation | Kerberos |

### Integration Framework

```
plugins/                # Plugin system
├── pypykatz_rekall.py  # Rekall plugin integration
└── (extensible)        # Future plugin support

parsers/                # File format parsers
├── cmdhelper.py        # Parser command interface
└── (various parsers)   # Extensible parser framework
```

## Development Roadmap and Feature Evolution

### Version History and Milestones

| Version | Key Features | Architecture Changes | Performance Improvements |
|---------|--------------|---------------------|-------------------------|
| **0.1.x** | Basic LSASS parsing | Monolithic structure | Initial implementation |
| **0.2.x** | Registry support | Modular packages | Template-based parsing |
| **0.3.x** | Remote capabilities | Network modules | Streaming analysis |
| **0.4.x** | DPAPI support | Plugin architecture | Memory optimization |
| **0.5.x** | Cloud credentials | Modern auth packages | Cross-platform stability |
| **Current** | Full feature set | Mature architecture | Production ready |

### Future Development Areas

#### Planned Enhancements
```
Roadmap (Estimated)
├── Enhanced Cloud Support
│   ├── Microsoft 365 credentials
│   ├── Azure Key Vault integration
│   └── Modern authentication flows
├── Performance Optimizations
│   ├── Multi-threading support
│   ├── Memory usage optimization
│   └── Faster pattern matching
├── Extended Platform Support
│   ├── ARM64 architecture
│   ├── Container environments
│   └── Cloud-native deployment
└── Advanced Analysis Features
    ├── Timeline analysis
    ├── Credential correlation
    └── Attack path mapping
```

#### Potential New Modules
| Module | Purpose | Complexity | Priority |
|--------|---------|------------|----------|
| **Timeline** | Credential usage timeline | High | Medium |
| **Correlation** | Cross-credential analysis | High | Medium |
| **ML Detection** | Anomaly detection | Very High | Low |
| **Blockchain** | Cryptocurrency wallet analysis | Medium | Low |
| **Mobile** | Mobile device credential extraction | Very High | Low |

## Architecture Strengths and Design Patterns

### Design Principles

#### 1. Separation of Concerns
```python
# Data Access Layer (What to read)
reader = MinidumpReader(filename)

# Template Layer (How to interpret)
template = MsvTemplate.get_template(sysinfo)

# Decryption Layer (How to decrypt)
decryptor = MsvDecryptor(reader, template, lsa_decryptor)

# Output Layer (How to present)
results = decryptor.to_json()
```

#### 2. Template-Based Architecture
```python
# Windows version-specific templates
class MsvTemplate_NT6:
    signature = b'\x83\x64\x24\x30\x00\x48\x8d\x45\xe0'
    first_entry_offset = -4
    list_entry = PKIWI_MSV1_0_LIST
```

#### 3. Pluggable Reader System
```python
# Unified interface for different data sources
class Reader:
    def read(self, size): pass
    def seek(self, position): pass
    def find_in_module(self, module, pattern): pass
```

### Architectural Benefits

| Benefit | Implementation | Impact |
|---------|----------------|--------|
| **Modularity** | Package-based structure | Easy maintenance |
| **Extensibility** | Plugin architecture | New features without core changes |
| **Cross-Platform** | Abstracted readers | Analysis on any OS |
| **Maintainability** | Clear separation of concerns | Easier debugging |
| **Testability** | Modular components | Unit testing possible |

## Performance Characteristics by Module

### Memory Usage Patterns

| Module | Memory Usage | Scaling | Optimization |
|--------|--------------|---------|--------------|
| **LSASS Decryptor** | Low (streaming) | O(1) | Template caching |
| **Registry Parser** | Low (file-based) | O(1) | Lazy loading |
| **Memory Dump** | Medium (buffered) | O(log n) | Chunk processing |
| **DPAPI Handler** | Low (on-demand) | O(1) | Key caching |
| **Network Modules** | Medium (buffering) | O(1) | Connection pooling |

### CPU Utilization Patterns

| Operation | CPU Pattern | Bottleneck | Optimization Potential |
|-----------|-------------|------------|----------------------|
| **Pattern Matching** | CPU-intensive | Single-threaded | Multi-threading possible |
| **Decryption** | Crypto-intensive | Algorithm efficiency | Hardware acceleration |
| **File I/O** | I/O bound | Storage speed | Async I/O possible |
| **Network Operations** | Network bound | Bandwidth/latency | Connection optimization |

## Conclusion

pypykatz represents a mature, well-architected credential extraction framework with:

### Architectural Strengths
- **Modular Design**: Clear separation between data access, processing, and output
- **Extensible Framework**: Plugin system allows new capabilities without core changes
- **Cross-Platform Compatibility**: Abstracted readers enable analysis on any platform
- **Template-Based Parsing**: Version-specific templates handle Windows evolution
- **Comprehensive Coverage**: Supports all major Windows authentication packages

### Technical Excellence
- **Linear Complexity**: O(n) scaling for most operations
- **Memory Efficient**: Streaming analysis prevents memory exhaustion
- **Production Ready**: Stable, tested, and actively maintained
- **Standards Compliant**: Follows established security research practices

### Strategic Position
pypykatz has evolved from a simple mimikatz port to a comprehensive credential analysis platform, suitable for penetration testing, incident response, digital forensics, and security research. Its architecture supports continued evolution while maintaining backward compatibility and cross-platform operation.

The framework's design enables security professionals to adapt to new Windows versions, authentication methods, and analysis requirements while maintaining a consistent, powerful interface for credential extraction and analysis.
