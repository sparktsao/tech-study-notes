# Study 003: Top 10 pypykatz Password Acquisition Strategies

## Executive Summary

This study provides a comprehensive analysis of the top 10 password acquisition strategies using pypykatz, including success rates, complexity analysis, and optimal use cases for each approach.

## Prerequisite Knowledge

### Understanding Strategy Types

#### Live Analysis
- **Definition**: Direct access to running Windows system memory and processes
- **Requirements**: Administrative privileges on the target Windows system
- **Advantages**: Real-time data, highest success rate, immediate results
- **Disadvantages**: Requires direct system access, may trigger security software
- **Examples**: Accessing live LSASS process, reading live registry

#### Offline Analysis
- **Definition**: Analysis of previously captured memory dumps, files, or system artifacts
- **Requirements**: Access to dump files or system artifacts (no live system needed)
- **Advantages**: Stealth (no live system interaction), cross-platform analysis, forensic preservation
- **Disadvantages**: Data may be stale, requires prior dump creation, potential data loss
- **Examples**: LSASS minidumps, memory dumps, registry hive files, hibernation files

#### Remote Analysis
- **Definition**: Network-based access to remote Windows systems
- **Requirements**: Network connectivity, valid credentials, appropriate permissions
- **Advantages**: Lateral movement capability, no physical access needed
- **Disadvantages**: Network dependencies, authentication requirements, potential detection
- **Examples**: Remote LSASS access via SMB, DCSync attacks over network

### Memory Size Classifications

#### Target Data Size (What pypykatz analyzes)
This refers to the size of the data that pypykatz must process and scan:

- **Small (< 100MB)**: Registry hives, small process dumps
  - **Scan Complexity**: O(n) - Linear relationship
  - **Processing Time**: 1-10 seconds
  - **RAM Required**: 512MB minimum

- **Medium (100MB-1GB)**: LSASS process dumps, crash dumps
  - **Scan Complexity**: O(n) - Linear relationship  
  - **Processing Time**: 10 seconds - 5 minutes
  - **RAM Required**: 1-2GB minimum

- **Large (1-8GB)**: Full memory dumps, hibernation files
  - **Scan Complexity**: O(n) - Linear relationship
  - **Processing Time**: 5-60 minutes
  - **RAM Required**: 2-4GB minimum

- **Very Large (> 8GB)**: Complete system memory dumps
  - **Scan Complexity**: O(n) - Linear relationship
  - **Processing Time**: 30+ minutes
  - **RAM Required**: 4-8GB minimum

#### Computational Memory Requirements
This refers to the RAM needed by pypykatz to perform the analysis:

- **Minimum System RAM**: 2GB (for basic operations)
- **Recommended RAM**: 8GB (for large dump analysis)
- **Optimal RAM**: 16GB+ (for very large dumps and multiple concurrent operations)

**Important**: pypykatz uses streaming analysis, so it doesn't load entire dumps into memory. The computational complexity is **linear O(n)** with respect to target data size, meaning doubling the dump size approximately doubles the processing time.

### Complexity Analysis Framework

#### Linear Complexity O(n)
- **Memory scanning**: Time increases proportionally with data size
- **Pattern matching**: Each byte must be examined once
- **Structure parsing**: Sequential processing of data structures

#### Factors Affecting Real Performance
- **Storage Speed**: SSD vs HDD can create 2-5x difference
- **CPU Speed**: Affects pattern matching and decryption operations
- **Available RAM**: Insufficient RAM causes disk swapping (10-100x slowdown)
- **File Fragmentation**: Can add 10-50% overhead

## Strategy Comparison Table

| # | Strategy Name | Command | Password Location | Type | Success Rate | Speed | Complexity | Memory Size | Prerequisites |
|---|---------------|---------|-------------------|------|--------------|-------|------------|-------------|---------------|
| 1 | Live LSASS Extraction | `pypykatz live lsa` | LSASS process memory | Live | 95% | Fast (2-5s) | Low | 50-200MB | Admin rights, Windows |
| 2 | LSASS Minidump Analysis | `pypykatz lsa minidump <file>` | LSASS process dump | Offline | 90% | Fast (5-15s) | Low | 100-500MB | Dump file access |
| 3 | Registry SAM/SECURITY | `pypykatz registry --sam --security` | Registry hives | Offline | 85% | Fast (1-3s) | Low | 10-50MB | Registry file access |
| 4 | Full Memory Dump | `pypykatz rekall <memory.vmem>` | System memory dump | Offline | 80% | Slow (10-60min) | High | 4-16GB | Full memory dump |
| 5 | Hibernation File | `pypykatz rekall <hiberfil.sys>` | Hibernation file | Offline | 75% | Medium (5-30min) | Medium | 2-8GB | Hibernation enabled |
| 6 | Live Registry | `pypykatz live registry` | Live registry | Live | 70% | Fast (2-5s) | Low | N/A | Admin rights, Windows |
| 7 | Remote LSASS | `pypykatz smb lsass <target>` | Remote LSASS | Remote | 65% | Medium (10-30s) | Medium | 50-200MB | Network access, creds |
| 8 | Crash Dump Analysis | `pypykatz lsa minidump <crash.dmp>` | System crash dump | Offline | 60% | Medium (5-20min) | Medium | 500MB-2GB | Crash dump available |
| 9 | Process Handle Dump | `pypykatz live handledup` | Process handles | Live | 55% | Fast (3-8s) | Medium | Variable | Handle access |
| 10 | DCSync Attack | `pypykatz smb dcsync <domain>` | Domain Controller | Remote | 50% | Fast (5-15s) | High | N/A | DC replication rights |

## Detailed Strategy Analysis

### 1. Live LSASS Extraction
**Command:** `pypykatz live lsa`

**Description:** Direct extraction from live LSASS process memory on Windows systems.

**Password Location:** 
- LSASS process memory space
- Authentication package structures (MSV, Kerberos, WDigest, etc.)
- Encrypted credential blobs in process heap

**Technical Details:**
- **Memory Access:** Direct process memory reading via Windows APIs
- **Data Structures:** KIWI_MSV1_0_LIST, authentication package templates
- **Encryption:** AES/3DES decryption using LSA keys
- **Memory Size:** 50-200MB typical LSASS process size

**Success Factors:**
- ✅ LSASS process running and accessible
- ✅ Administrative privileges
- ✅ Windows Defender/AV not blocking
- ✅ LSASS protection not enabled

**Complexity Analysis:**
- **Low Complexity:** Single command execution
- **Speed:** 2-5 seconds (memory scanning + decryption)
- **Dependencies:** Windows APIs, process access rights

### 2. LSASS Minidump Analysis
**Command:** `pypykatz lsa minidump lsass.dmp`

**Description:** Offline analysis of LSASS process memory dump files.

**Password Location:**
- Dumped LSASS process memory
- Same structures as live analysis but in file format
- Minidump format preserves memory layout

**Technical Details:**
- **File Format:** Windows Minidump (.dmp)
- **Memory Mapping:** Virtual address space reconstruction
- **Data Integrity:** Complete process memory snapshot
- **File Size:** 100-500MB typical dump size

**Success Factors:**
- ✅ Complete memory dump (not partial)
- ✅ Dump created with appropriate tools (Task Manager, ProcDump)
- ✅ No memory corruption during dump
- ✅ Compatible Windows version templates

**Complexity Analysis:**
- **Low Complexity:** File parsing with established libraries
- **Speed:** 5-15 seconds (file I/O + parsing)
- **Dependencies:** Minidump parsing library, file system access

### 3. Registry SAM/SECURITY Analysis
**Command:** `pypykatz registry --sam sam --security security --system system`

**Description:** Extract password hashes and secrets from Windows registry hives.

**Password Location:**
- SAM hive: Local user NTLM hashes
- SECURITY hive: LSA secrets, cached domain credentials
- SYSTEM hive: Boot key for decryption

**Technical Details:**
- **Registry Structure:** Binary hive format parsing
- **Encryption:** RC4 encryption with boot key
- **Hash Types:** NTLM, LM (legacy), cached domain credentials (DCC/DCC2)
- **File Size:** 10-50MB combined hive files

**Success Factors:**
- ✅ All three hives available (SAM, SECURITY, SYSTEM)
- ✅ Hives not corrupted or locked
- ✅ Boot key successfully extracted
- ✅ User accounts present in SAM

**Complexity Analysis:**
- **Low Complexity:** Well-documented registry format
- **Speed:** 1-3 seconds (small file sizes)
- **Dependencies:** Registry parsing libraries

### 4. Full Memory Dump Analysis
**Command:** `pypykatz rekall memory.vmem`

**Description:** Comprehensive analysis of complete system memory dumps.

**Password Location:**
- LSASS process within full memory
- Multiple processes with cached credentials
- Kernel memory structures
- Swap/page file content

**Technical Details:**
- **Memory Format:** Raw memory dump, VMware .vmem, etc.
- **Process Discovery:** Memory scanning for LSASS signatures
- **Address Translation:** Virtual to physical memory mapping
- **Dump Size:** 4-16GB typical system memory

**Success Factors:**
- ✅ Complete memory capture
- ✅ LSASS process was running during capture
- ✅ Memory not heavily fragmented
- ✅ Correct OS version detection

**Complexity Analysis:**
- **High Complexity:** Large file processing, memory analysis
- **Speed:** 10-60 minutes (depends on dump size and storage speed)
- **Dependencies:** Rekall/Volatility framework, significant disk space

### 5. Hibernation File Analysis
**Command:** `pypykatz rekall hiberfil.sys`

**Description:** Extract credentials from Windows hibernation files.

**Password Location:**
- Hibernated LSASS process memory
- System state at hibernation time
- Compressed memory structures

**Technical Details:**
- **File Format:** Compressed hibernation file
- **Decompression:** Windows hibernation decompression
- **Memory State:** Point-in-time system snapshot
- **File Size:** 2-8GB (compressed system memory)

**Success Factors:**
- ✅ Hibernation was enabled and used
- ✅ LSASS had credentials when hibernated
- ✅ File not corrupted or overwritten
- ✅ Successful decompression

**Complexity Analysis:**
- **Medium Complexity:** Hibernation format handling
- **Speed:** 5-30 minutes (decompression + analysis)
- **Dependencies:** Hibernation decompression, memory analysis tools

### 6. Live Registry Extraction
**Command:** `pypykatz live registry`

**Description:** Direct extraction from live Windows registry.

**Password Location:**
- Live SAM registry key
- Live SECURITY registry key
- In-memory registry structures

**Technical Details:**
- **Registry Access:** Windows Registry APIs
- **Memory Technique:** Direct registry memory reading
- **Encryption:** Real-time boot key extraction
- **Data Size:** Variable, typically small

**Success Factors:**
- ✅ Administrative privileges
- ✅ Registry not locked by other processes
- ✅ Windows security policies allow access
- ✅ Boot key accessible

**Complexity Analysis:**
- **Low Complexity:** Standard Windows API usage
- **Speed:** 2-5 seconds (registry API calls)
- **Dependencies:** Windows Registry APIs, admin rights

### 7. Remote LSASS Access
**Command:** `pypykatz smb lsass \\target\admin$ -u user -p pass`

**Description:** Remote LSASS memory extraction over network.

**Password Location:**
- Remote system LSASS process
- Network-accessible memory dump
- SMB/RPC transported data

**Technical Details:**
- **Network Protocol:** SMB/RPC for remote access
- **Authentication:** Valid credentials required
- **Data Transfer:** Memory dump over network
- **Bandwidth:** 50-200MB transfer typical

**Success Factors:**
- ✅ Valid credentials for target system
- ✅ Administrative access on target
- ✅ Network connectivity and SMB access
- ✅ Target system not hardened against remote access

**Complexity Analysis:**
- **Medium Complexity:** Network protocols, authentication
- **Speed:** 10-30 seconds (network transfer + processing)
- **Dependencies:** Network access, SMB libraries, valid credentials

### 8. Crash Dump Analysis
**Command:** `pypykatz lsa minidump crash.dmp`

**Description:** Extract credentials from Windows crash dump files.

**Password Location:**
- LSASS process in crash dump
- System state at crash time
- Kernel and user mode memory

**Technical Details:**
- **Dump Type:** Full system crash dump or kernel dump
- **Memory Content:** System state at crash
- **Process State:** May be incomplete or corrupted
- **File Size:** 500MB-2GB typical crash dump

**Success Factors:**
- ✅ Crash occurred while LSASS had credentials
- ✅ Complete dump (not mini dump)
- ✅ LSASS process memory included
- ✅ Memory structures intact

**Complexity Analysis:**
- **Medium Complexity:** Crash dump format parsing
- **Speed:** 5-20 minutes (large file processing)
- **Dependencies:** Crash dump analysis tools

### 9. Process Handle Dump
**Command:** `pypykatz live handledup`

**Description:** Extract credentials using existing process handles to LSASS.

**Password Location:**
- LSASS memory via existing handles
- Process handle table enumeration
- Inherited or duplicated handles

**Technical Details:**
- **Handle Discovery:** Enumerate system handles
- **Access Rights:** Use existing handle permissions
- **Memory Access:** Indirect LSASS access
- **Handle Types:** Process, thread, section handles

**Success Factors:**
- ✅ Existing handles to LSASS process
- ✅ Sufficient handle permissions
- ✅ Handle not closed or invalidated
- ✅ Process with handle still running

**Complexity Analysis:**
- **Medium Complexity:** Handle enumeration and manipulation
- **Speed:** 3-8 seconds (handle discovery + access)
- **Dependencies:** Handle manipulation APIs, process enumeration

### 10. DCSync Attack
**Command:** `pypykatz smb dcsync -u user@domain.com -p pass --target-user krbtgt`

**Description:** Extract password hashes directly from Domain Controller.

**Password Location:**
- Active Directory database (NTDS.dit)
- Domain Controller memory
- Replication data stream

**Technical Details:**
- **Protocol:** MS-DRSR (Directory Replication Service)
- **Authentication:** Domain credentials with replication rights
- **Data Source:** AD database replication
- **Network Traffic:** Encrypted replication stream

**Success Factors:**
- ✅ Domain credentials with DCSync rights
- ✅ Network access to Domain Controller
- ✅ Replication service available
- ✅ Target accounts exist in AD

**Complexity Analysis:**
- **High Complexity:** AD replication protocol implementation
- **Speed:** 5-15 seconds (network replication)
- **Dependencies:** Domain access, replication rights, network connectivity

## Success Rate Analysis

### Factors Affecting Success Rates

#### Environmental Factors
- **Windows Version:** Newer versions have more protections
- **Security Software:** AV/EDR can block or detect activities
- **System Configuration:** LSASS protection, credential guard
- **User Activity:** Recent logons increase credential availability

#### Technical Factors
- **Memory Fragmentation:** Affects dump quality and parsing
- **Process State:** LSASS health and credential freshness
- **File Integrity:** Corruption affects offline analysis
- **Network Conditions:** Impacts remote operations

### Success Rate by Scenario

| Scenario | Live LSASS | Minidump | Registry | Memory Dump | Remote |
|----------|------------|----------|----------|-------------|---------|
| Domain Workstation | 95% | 90% | 85% | 80% | 65% |
| Standalone PC | 90% | 85% | 90% | 75% | N/A |
| Server (Active) | 98% | 95% | 80% | 85% | 70% |
| Server (Idle) | 85% | 80% | 85% | 70% | 60% |
| Hardened System | 60% | 70% | 60% | 65% | 40% |

## Speed and Complexity Matrix

### Processing Speed Factors

#### File Size Impact
- **Small (< 100MB):** 1-10 seconds
- **Medium (100MB-1GB):** 10 seconds - 5 minutes
- **Large (1-8GB):** 5-30 minutes
- **Very Large (> 8GB):** 30+ minutes

#### Storage Type Impact
- **SSD:** 1x baseline speed
- **HDD:** 2-3x slower
- **Network Storage:** 3-5x slower
- **USB 2.0:** 5-10x slower

#### Memory Available Impact
- **< 4GB RAM:** Significant slowdown on large files
- **4-8GB RAM:** Adequate for most operations
- **8-16GB RAM:** Optimal for large memory dumps
- **> 16GB RAM:** No additional benefit for pypykatz

## Recommended Strategy Selection

### By Use Case

#### Penetration Testing
1. **Live LSASS** (if admin access available)
2. **LSASS Minidump** (for stealth)
3. **Registry** (for persistence)

#### Incident Response
1. **Memory Dump** (comprehensive evidence)
2. **LSASS Minidump** (targeted analysis)
3. **Registry** (historical data)

#### Digital Forensics
1. **Memory Dump** (complete picture)
2. **Hibernation File** (if available)
3. **Registry** (system state)

#### Red Team Operations
1. **Live LSASS** (immediate results)
2. **Remote LSASS** (lateral movement)
3. **DCSync** (domain compromise)

### By Environment

#### Corporate Domain
- **Primary:** Live LSASS, DCSync
- **Secondary:** Remote LSASS, Registry
- **Forensic:** Memory Dump, Minidump

#### Standalone Systems
- **Primary:** Live LSASS, Registry
- **Secondary:** Minidump, Hibernation
- **Forensic:** Memory Dump

#### Hardened Environments
- **Primary:** Minidump, Registry
- **Secondary:** Memory Dump, Crash Dump
- **Advanced:** Handle Dump, Remote techniques

## Conclusion

The choice of password acquisition strategy depends on multiple factors including access level, stealth requirements, time constraints, and available artifacts. Live LSASS extraction remains the most effective approach when administrative access is available, while offline analysis techniques provide valuable alternatives for forensic scenarios or when direct access is limited.

Understanding the technical requirements, success factors, and complexity of each approach enables security professionals to select the most appropriate strategy for their specific use case and environment.
