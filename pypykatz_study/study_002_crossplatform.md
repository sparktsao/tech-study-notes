# Study 002: Cross-Platform Analysis - pypykatz Capabilities

## Executive Summary

This study clarifies what "cross-platform" means for pypykatz and addresses the common misconception about its capabilities when running on non-Windows systems like macOS and Linux.

**Key Clarifications:**
- pypykatz running on macOS/Linux can ONLY analyze Windows memory/data
- pypykatz CANNOT extract credentials from macOS or Linux systems
- "Cross-platform" refers to the analysis platform, not the target platform
- The target data must always be from Windows systems

## Understanding Cross-Platform Analysis

### What pypykatz CAN Do on macOS/Linux

#### 1. Analyze Windows Memory Dumps
```
Scenario: Windows system memory dump analysis on macOS
- Source: Windows 10 system memory dump (4GB .vmem file)
- Analysis Platform: macOS with pypykatz
- Target: Extract Windows credentials from the dump
- Result: ✓ WORKS - Can extract NTLM hashes, Kerberos tickets, etc.
```

#### 2. Process Windows Minidumps
```
Scenario: LSASS minidump analysis on Linux
- Source: lsass.exe process dump from Windows server
- Analysis Platform: Linux with pypykatz
- Target: Extract stored Windows credentials
- Result: ✓ WORKS - Full credential extraction capability
```

#### 3. Parse Windows Registry Hives
```
Scenario: Offline Windows registry analysis on macOS
- Source: SAM, SECURITY, SYSTEM hives from Windows
- Analysis Platform: macOS with pypykatz
- Target: Extract cached domain credentials, LSA secrets
- Result: ✓ WORKS - Complete registry credential extraction
```

#### 4. Analyze Hibernation Files
```
Scenario: Windows hiberfil.sys analysis on Linux
- Source: hiberfil.sys from Windows laptop
- Analysis Platform: Linux with pypykatz + volatility
- Target: Extract credentials from hibernated memory
- Result: ✓ WORKS - Can process hibernation file data
```

### What pypykatz CANNOT Do on macOS/Linux

#### 1. Extract macOS Credentials
```
Scenario: Attempt to extract macOS keychain passwords
- Source: macOS system with Keychain
- Analysis Platform: macOS with pypykatz
- Target: Extract macOS user passwords
- Result: ✗ IMPOSSIBLE - pypykatz only understands Windows structures
```

#### 2. Analyze Linux Memory
```
Scenario: Attempt to extract Linux user credentials
- Source: Linux system memory dump
- Analysis Platform: Linux with pypykatz
- Target: Extract /etc/shadow hashes or Kerberos tickets
- Result: ✗ IMPOSSIBLE - Different memory structures and authentication systems
```

#### 3. Live Memory Access on Non-Windows
```
Scenario: Attempt live credential extraction on macOS
- Source: Live macOS system
- Analysis Platform: macOS with pypykatz
- Target: Extract current user credentials
- Result: ✗ IMPOSSIBLE - No equivalent to Windows LSASS process
```

## Technical Architecture Analysis

### Windows-Specific Components in pypykatz

#### 1. LSASS Process Structure Knowledge
```python
# From pypykatz source - Windows-specific structures
class KIWI_MSV1_0_LIST:
    """Windows LSASS MSV authentication package structure"""
    def __init__(self):
        self.Flink = None           # Windows LIST_ENTRY
        self.Blink = None           # Windows LIST_ENTRY
        self.LocallyUniqueIdentifier = None  # Windows LUID
        self.UserName = None        # Windows UNICODE_STRING
        self.Domaine = None         # Windows UNICODE_STRING
```

#### 2. Windows Authentication Package Templates
```python
# pypykatz only knows Windows authentication packages:
- MSV1_0 (NTLM)
- Kerberos (Windows implementation)
- WDigest (Windows-specific)
- SSP/SSPI (Windows Security Support Provider)
- TsPkg (Terminal Services Package)
- CredMan (Windows Credential Manager)
- DPAPI (Windows Data Protection API)
```

#### 3. Windows Registry Structure Knowledge
```python
# Windows-specific registry parsing
class SAMHashes:
    """Extracts NTLM hashes from Windows SAM registry"""
    
class LSASecrets:
    """Extracts LSA secrets from Windows SECURITY registry"""
```

### Why Other Operating Systems Are Not Supported

#### macOS Authentication Architecture
- **Keychain Services**: Different API and storage format
- **Open Directory**: Different authentication framework
- **Security Framework**: Different credential storage
- **Memory Layout**: Different process memory organization

#### Linux Authentication Architecture
- **PAM (Pluggable Authentication Modules)**: Different authentication system
- **/etc/shadow**: Different password storage format
- **Kerberos**: Different implementation and memory structures
- **SSSD**: Different credential caching mechanism

## Practical Use Cases for Cross-Platform Analysis

### Digital Forensics Workflow
```
1. Incident occurs on Windows server
2. Memory dump created using WinPmem or similar
3. Dump file transferred to Linux forensics workstation
4. Analyst uses pypykatz on Linux to extract Windows credentials
5. Results integrated with other forensic tools
```

### Penetration Testing Scenario
```
1. Compromise Windows workstation during pentest
2. Create LSASS minidump using task manager
3. Transfer dump to macOS laptop
4. Use pypykatz on macOS to extract credentials offline
5. Use extracted credentials for lateral movement
```

### Incident Response
```
1. Windows server suspected of compromise
2. Create memory snapshot for analysis
3. Analyze on isolated Linux system using pypykatz
4. Extract credentials to understand attack scope
5. Generate report without touching production Windows systems
```

## Comparison with Native Tools

### macOS Credential Extraction Tools
- **Keychain-Dump**: Extracts macOS keychain passwords
- **OSXCollector**: Collects macOS forensic artifacts
- **mac_apt**: macOS artifact parsing tool

### Linux Credential Extraction Tools
- **mimipenguin**: Extracts Linux passwords from memory
- **LaZagne**: Multi-platform password recovery
- **hashcat**: Password hash cracking

### Windows-Only Tools (Like Original mimikatz)
- **mimikatz**: Windows-only, live system access
- **Invoke-Mimikatz**: PowerShell version, Windows-only
- **SafetyKatz**: .NET version, Windows-only

## File Format Support Matrix

| File Type | Source OS | Analysis Platform | pypykatz Support |
|-----------|-----------|-------------------|------------------|
| LSASS Minidump | Windows | Windows/Linux/macOS | ✓ Full Support |
| Memory Dump (.vmem) | Windows | Windows/Linux/macOS | ✓ Full Support |
| Hibernation File | Windows | Windows/Linux/macOS | ✓ Full Support |
| Registry Hives | Windows | Windows/Linux/macOS | ✓ Full Support |
| Crash Dump | Windows | Windows/Linux/macOS | ✓ Full Support |
| macOS Memory Dump | macOS | Any | ✗ No Support |
| Linux Core Dump | Linux | Any | ✗ No Support |
| VMware .vmem | Windows VM | Windows/Linux/macOS | ✓ Full Support |
| VirtualBox .sav | Windows VM | Windows/Linux/macOS | ✓ Full Support |

## Installation and Dependencies

### pypykatz on Different Platforms

#### Windows Installation
```bash
pip install pypykatz
# Can perform live analysis + dump analysis
```

#### macOS Installation
```bash
pip install pypykatz
# Can only perform dump analysis of Windows data
```

#### Linux Installation
```bash
pip install pypykatz
# Can only perform dump analysis of Windows data
```

### Platform-Specific Limitations

#### Live Analysis Capabilities
| Feature | Windows | macOS | Linux |
|---------|---------|-------|-------|
| Live LSASS Access | ✓ | ✗ | ✗ |
| Live Registry Access | ✓ | ✗ | ✗ |
| Process Manipulation | ✓ | ✗ | ✗ |
| Token Impersonation | ✓ | ✗ | ✗ |

#### Offline Analysis Capabilities
| Feature | Windows | macOS | Linux |
|---------|---------|-------|-------|
| Minidump Analysis | ✓ | ✓ | ✓ |
| Memory Dump Analysis | ✓ | ✓ | ✓ |
| Registry Hive Analysis | ✓ | ✓ | ✓ |
| Hibernation File Analysis | ✓ | ✓ | ✓ |

## Common Misconceptions

### Misconception 1: "Universal Credential Extractor"
**Wrong**: pypykatz can extract credentials from any operating system
**Correct**: pypykatz only extracts Windows credentials, regardless of analysis platform

### Misconception 2: "Cross-Platform Live Analysis"
**Wrong**: pypykatz can perform live analysis on macOS/Linux
**Correct**: Live analysis only works on Windows; other platforms limited to dump analysis

### Misconception 3: "Native macOS/Linux Support"
**Wrong**: pypykatz has native support for macOS/Linux credential systems
**Correct**: pypykatz only understands Windows authentication structures

## Conclusion

### What "Cross-Platform" Really Means for pypykatz

**Cross-Platform Analysis Environment:**
- pypykatz can RUN on Windows, macOS, and Linux
- The analysis platform is flexible

**Windows-Only Target Data:**
- pypykatz can ONLY analyze Windows credential data
- The target system must always be Windows

### Practical Benefits

1. **Forensic Flexibility**: Analyze Windows evidence on any platform
2. **Operational Security**: Perform analysis on isolated non-Windows systems
3. **Tool Integration**: Integrate with Linux/macOS forensic workflows
4. **Development Environment**: Develop and test on preferred platform

### Key Takeaway

When we say pypykatz is "cross-platform," we mean:
- **Analysis Platform**: Can run on Windows, macOS, Linux
- **Target Platform**: Can only analyze Windows systems

This is fundamentally different from tools that can extract credentials from multiple operating systems. pypykatz is a Windows credential extraction tool that happens to run on multiple platforms for analysis convenience.

The cross-platform capability is about **where you run the analysis**, not **what systems you can analyze**.
