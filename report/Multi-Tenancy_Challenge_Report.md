# Cloud Forensics Investigation Report

**Title:** Forensic Challenges in a Multi-Tenant Cloud Environment

**Student Name:** Komaleswari S

**Register Number:** ____________________

**Assigned Topic:** Multi-Tenancy Challenge

---

## 1. Introduction

Cloud computing has transformed the way organizations store, process, and manage data by offering on-demand, scalable, and cost-effective services. A defining characteristic of the public cloud is **multi-tenancy**, where a single physical infrastructure — servers, storage, networks, and even individual database instances — is shared simultaneously by many independent customers, called *tenants*.

While multi-tenancy delivers efficiency and lower costs, it introduces serious complications for **digital forensics**. When an incident occurs and evidence belonging to one tenant must be collected, investigators face the difficulty of extracting that evidence from an environment physically and logically intermingled with the data of hundreds of other, uninvolved organizations. This report examines the forensic challenges created by multi-tenancy, using a scenario in which a cloud provider hosts data for 100 organizations on shared infrastructure and evidence relating to only one of them is required.

## 2. Case Scenario

A Cloud Service Provider (CSP) operates a large **Infrastructure-as-a-Service / Software-as-a-Service** platform. On a shared cluster of physical hosts, the CSP hosts the data and workloads of **100 different organizations (tenants)**. Each tenant's data is logically separated using virtualization, containerization, and database-level partitioning, but the underlying CPU, memory, disk, and network hardware are **shared**.

One tenant — *Organization-A* — is the subject of a legal investigation (suspected financial fraud and data exfiltration). A court order authorizes investigators to collect digital evidence relating **only to Organization-A**. The forensic team must acquire relevant logs, virtual machine images, storage volumes, and database records **without accessing, exposing, or altering the data of the other 99 tenants** who are not part of the investigation.

## 3. Problem Identification

The central problem is that **evidence for a single tenant is physically co-mingled with the data of many unrelated tenants** on shared resources. This creates the following specific problems:

- **Data commingling:** A single physical disk, memory page, or log file may contain data from dozens of tenants at once.
- **Isolation vs. accessibility:** The logical isolation that protects tenants also prevents investigators from cleanly "carving out" only the relevant evidence.
- **Privacy and legal exposure:** Any bulk acquisition (e.g., imaging a whole physical disk) risks capturing the private data of innocent tenants, breaching confidentiality and data-protection laws.
- **Chain of custody:** Proving that acquired evidence belongs solely to Organization-A and was not contaminated by other tenants' data is difficult.
- **Dependence on the provider:** Investigators usually cannot access the physical hardware directly and must rely on the CSP's cooperation and tooling.

## 4. Digital Evidence

Potential sources of digital evidence relevant to Organization-A in this multi-tenant environment include:

| Evidence Type | Description | Multi-Tenancy Concern |
|---------------|-------------|-----------------------|
| Virtual Machine (VM) images / snapshots | Disk and memory state of the tenant's instances | Reside on shared hypervisor hosts |
| Storage volumes / object storage | Files, databases, blobs owned by the tenant | Stored in shared storage pools |
| Application & access logs | Authentication, API calls, user activity | Log files often shared across tenants |
| Database records | Rows/tables specific to the tenant | May share a physical DB instance/table |
| Network traffic / flow logs | Communication to and from tenant workloads | Captured on shared network devices |
| Hypervisor / host logs | VM lifecycle, resource allocation events | Contain entries for all co-located tenants |
| Cloud audit trails (e.g., CloudTrail) | Control-plane actions on tenant resources | Filtered per account/subscription |

## 5. Investigation Procedure

1. **Legal authorization:** Obtain a court order / warrant scoped strictly to Organization-A and engage the CSP under a defined legal process (e.g., Service Level Agreement clauses, subpoena).
2. **Identify tenant boundaries:** Work with the CSP to map which accounts, subscription IDs, VM instances, storage buckets, and database schemas belong to Organization-A.
3. **Targeted (logical) acquisition:** Instead of imaging shared physical media, acquire evidence at the **logical/tenant level** — snapshot the tenant's specific VMs, export the tenant's storage buckets, and query only the tenant's database partitions.
4. **Isolate and hash:** Copy the acquired data to a forensically sound container, compute cryptographic hashes (SHA-256) immediately to preserve integrity.
5. **Filter and verify tenant ownership:** Use account identifiers, tags, and access-control metadata to confirm every artifact belongs only to Organization-A; redact or discard anything belonging to other tenants.
6. **Maintain chain of custody:** Document who acquired what, when, from which cloud region/service, and how, with CSP attestation.
7. **Analysis:** Examine the isolated evidence in a controlled forensic workstation using appropriate tools.
8. **Reporting:** Produce findings that are court-admissible and demonstrably free of other tenants' data.

## 6. Challenges

Multi-tenancy creates the following core forensic challenges:

- **Multi-tenancy itself:** One infrastructure serves many customers, so no single tenant "owns" the hardware from which evidence must be extracted.
- **Shared resources:** CPU, RAM, disk, and network are pooled; physical acquisition inevitably sweeps up multiple tenants' data.
- **Evidence isolation:** Cleanly separating one tenant's evidence from co-mingled data is technically hard and must be proven.
- **Privacy:** Collecting evidence must not violate the confidentiality or data-protection rights (e.g., GDPR) of the other 99 tenants.
- **Data separation:** Logical separation (VMs, containers, DB partitions) helps but is invisible at the physical layer and can be imperfect.
- **Evidence collection:** Investigators cannot touch the hardware directly and depend on the CSP's cooperation, APIs, and trustworthiness.
- **Additional issues:** Data volatility, cross-border data location/jurisdiction, weak or shared logging, and difficulty maintaining chain of custody.

## 7. Cloud Forensic Tools

- **AWS CloudTrail / Azure Monitor / Google Cloud Audit Logs** — per-tenant (per-account) control-plane audit trails for scoped log collection.
- **FTK (Forensic Toolkit) & EnCase** — analysis of acquired disk/volume images with hashing and reporting.
- **Magnet AXIOM Cloud** — acquisition and analysis of cloud-based evidence from major providers.
- **The Sleuth Kit (TSK) / Autopsy** — open-source examination of file systems from exported volumes.
- **Cellebrite / Oxygen (cloud modules)** — cloud data extraction with account-scoped access.
- **Provider snapshot & export APIs** (e.g., AWS EBS snapshots, S3 export) — for targeted, tenant-scoped logical acquisition.
- **Volatility** — memory forensics on tenant VM memory captures.

## 8. Investigation Flowchart

```
        ┌─────────────────────────────┐
        │  Incident reported for      │
        │  Organization-A (1 of 100)  │
        └──────────────┬──────────────┘
                       ▼
        ┌─────────────────────────────┐
        │  Obtain legal authorization │
        │  scoped to Organization-A   │
        └──────────────┬──────────────┘
                       ▼
        ┌─────────────────────────────┐
        │  Engage CSP & map tenant    │
        │  boundaries (accounts/IDs)  │
        └──────────────┬──────────────┘
                       ▼
        ┌─────────────────────────────┐
        │  Targeted LOGICAL           │
        │  acquisition (tenant only)  │
        └──────────────┬──────────────┘
                       ▼
        ┌─────────────────────────────┐
        │  Hash + isolate evidence;   │
        │  verify tenant ownership    │
        └──────────────┬──────────────┘
                       ▼
        ┌─────────────────────────────┐
        │  Filter out other tenants'  │
        │  data (privacy protection)  │
        └──────────────┬──────────────┘
                       ▼
        ┌─────────────────────────────┐
        │  Analyze with forensic tools│
        │  (maintain chain of custody)│
        └──────────────┬──────────────┘
                       ▼
        ┌─────────────────────────────┐
        │  Court-admissible report    │
        └─────────────────────────────┘
```

## 9. Conclusion

Multi-tenancy is the backbone of the modern public cloud, but it is one of the hardest problems in cloud forensics. Because the infrastructure is shared among many organizations, investigators cannot simply seize a hard drive; doing so would violate the privacy of uninvolved tenants and compromise the admissibility of evidence. The key to overcoming the multi-tenancy challenge is to shift from **physical acquisition to targeted, logical, tenant-scoped acquisition** — collecting only the accounts, snapshots, storage, and logs that belong to the organization under investigation, verifying ownership, and preserving a defensible chain of custody. Success depends heavily on cooperation with the Cloud Service Provider, strong per-tenant logging, and forensic tools designed for cloud environments.

## 10. References

1. Ruan, K., Carthy, J., Kechadi, T., & Crosbie, M. (2011). *Cloud forensics.* In Advances in Digital Forensics VII, IFIP AICT, Springer. https://doi.org/10.1007/978-3-642-24212-0_3
2. NIST. (2014). *NIST Cloud Computing Forensic Science Challenges (NIST IR 8006).* National Institute of Standards and Technology. https://csrc.nist.gov/publications/detail/nistir/8006/draft
3. Simou, S., Kalloniatis, C., Kavakli, E., & Gritzalis, S. (2014). *Cloud Forensics: Identifying the Major Issues and Challenges.* In Advanced Information Systems Engineering (CAiSE), Springer. https://doi.org/10.1007/978-3-319-07881-6_19
