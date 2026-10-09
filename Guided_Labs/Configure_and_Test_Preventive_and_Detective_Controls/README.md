# Configure and Test Preventive and Detective Security Controls

![Windows Server](https://img.shields.io/badge/Windows_Server_2019-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![Active Directory](https://img.shields.io/badge/Active_Directory-0078D6?style=for-the-badge&logo=microsoft&logoColor=white)
**Windows Event Viewer** | **Local Security Policy** | **NTFS/SMB Permissions**

## Scenario & Objective
Closed an over-permissive SMB share with a preventive control (ACL hardening on `TOOLS`), then enabled and validated a detective control (object access auditing on `C:\LABFILES`) in a Windows Server 2019 domain.

## Execution Methodology

### Phase 1: Preventive Control (Restrict the `TOOLS` Share)
**Baseline test (unwanted activity):**
- Signed in to PC10 as a non-administrative domain user (`Sam`) and browsed to `\\10.1.16.1\TOOLS` on DC10. The share contents were visible, which violated the requirement that only domain and local administrators have access.

**Root cause:** the `Domain Users` group inherited Read permissions from the parent folder, and `Everyone` held an entry on the ACL.

**Remediation on DC10 (Server Manager > File and Storage Services > Shares > `TOOLS` > Properties > Permissions > Customize permissions):**
1. Disabled inheritance and selected **Convert inherited permissions into explicit permissions**.
2. Removed both `structureality\Users` entries.
3. Removed the `Everyone` entry.
4. Left `Domain Admins` and `LocalAdmin` as the only principals with access.

**Design decision:** relied on Windows' implicit deny instead of an explicit Deny ACE. Administrators are also members of `Domain Users` and `Everyone`, so an explicit Deny on those groups would lock out the administrators who need access.

**Validation:**

| Account | Group Membership | Result after change |
|---|---|---|
| `Sam` | Standard domain user | Network error, access denied |
| `Jaime` | `LocalAdmin` | Share accessible |

**Control classification:** file permissions are a *Technical* control with *Preventive* function.

### Phase 2: Detective Control (Audit Object Access)
**Baseline test (unlogged activity):**
- Signed in to PC10 as `Jaime` (`LocalAdmin`) and deleted `C:\LABFILES\empty`.
- Searched the Security log in Event Viewer for `empty`. No record existed, confirming that folder deletion was not audited.

**Configuration:**
1. Enabled the audit policy switch: *Local Security Policy > Local Policies > Audit Policy > Audit object access*, with **Success** and **Failure** both checked.
2. Added an on-object audit entry (SACL) on `C:\LABFILES`: *Properties > Security > Advanced > Auditing > Add*, principal `Everyone`, advanced permissions **Delete subfolders and files** and **Delete**, with **Replace all child object auditing entries** enabled.

**Validation:**
- Deleted `C:\LABFILES\pcaps` as `Jaime`.
- In Event Viewer (*Windows Logs > Security*), located Event ID **4660** ("An object was deleted"), then the associated Event ID **4663** about five records above it. The 4663 record shows `Object Name: C:\LABFILES\pcaps`, which proves what was deleted.

| Event ID | Meaning | Contains object name |
|---|---|---|
| 4660 | An object was deleted | No |
| 4663 | An attempt was made to access an object | Yes |

**Control classification:** audit logging is a *Technical* control with *Detective* function.

## Enterprise Remediation
1. **Least privilege on shares:** grant access through role-based security groups (AGDLP pattern) and remove `Everyone` and `Users` from sensitive ACLs. Review both share-level and NTFS permissions, since the most restrictive set applies.
2. **Break inheritance deliberately:** for restricted folders, disable inheritance with explicit conversion, then prune entries. Avoid explicit Deny ACEs except for narrow, tested exceptions.
3. **Scope auditing with SACLs:** audit only sensitive paths and actions (delete, write, permission change) to limit log volume.
4. **Use Advanced Audit Policy Configuration** (*Object Access > Audit File System*) through GPO instead of the legacy local policy, so auditing is consistent across the domain.
5. **Centralize logs:** forward Security events 4660 and 4663 to a SIEM, alert on deletions in protected paths, and size the Security log to avoid overwrite.
6. **Report variances:** when a configuration deviates from policy or baseline, report it to the security team with recommended remediation.

## Lessons Learned
- Event 4660 records that a deletion happened but not the object name. Correlate it with the adjacent Event 4663 to identify the object.
- Enabling *Audit object access* alone logs nothing for most file objects; an on-object SACL is also required.
- If the deletion event does not appear after configuring auditing, restart the host and repeat the test.
- Convert inherited permissions before removing entries; otherwise the inherited entries cannot be edited.

## Skills & Technologies
Preventive Controls, Detective Controls, Access Control Lists, NTFS Permissions, SMB Share Security, Least Privilege, Active Directory, Domain Admins, Security Auditing, SACL, Windows Event Viewer, Event ID 4663, Event ID 4660, Local Security Policy, Windows Server 2019, Log Analysis
