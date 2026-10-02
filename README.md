# Cisco Enterprise Network Segmentation & ACL Security Lab

A Cisco Packet Tracer security lab demonstrating enterprise network segmentation, inter-VLAN routing, and access control using extended ACLs.

The lab simulates a small enterprise environment with separate Admin, Sales, IT, and Payroll networks. Security controls are implemented to restrict access to sensitive resources while maintaining required business connectivity.

The project includes VLAN segmentation, router-on-a-stick inter-VLAN routing, baseline security testing, ACL-based access control, and controlled validation of authorized and unauthorized access.

## Objectives

- Segment the enterprise network using VLANs.
- Configure inter-VLAN routing using router-on-a-stick.
- Establish baseline connectivity between departmental networks.
- Identify the security risks of unrestricted inter-VLAN communication.
- Implement an extended ACL to restrict Sales access to sensitive networks.
- Maintain required access for Admin and IT users.
- Validate authorized and unauthorized access through controlled security testing.
- Demonstrate how network segmentation and ACLs support least-privilege access.

## Scenario

This lab simulates a small enterprise branch network for a financial services organization.

The environment contains separate networks for:

- Admin — administrative users requiring access to sensitive resources.
- Sales — users who require access to IT support but should not access Admin systems or the Payroll resource.
- IT — technical staff requiring broader network access for support and administration.
- Payroll — a sensitive resource representing a cloud-hosted payroll system.

The security requirement is to separate these departments using VLANs and control access between them using an extended ACL.

The lab first establishes unrestricted inter-VLAN connectivity to demonstrate the security risk. An ACL is then implemented to restrict Sales access to the Admin and Payroll networks while maintaining required access to IT, Admin, and Payroll resources for authorized departments.

## Network Architecture

The lab uses a Cisco 2911 router and a Cisco 2960 switch to provide VLAN segmentation and inter-VLAN routing.

### Devices

- 1 × Cisco 2911 Router — R1
- 1 × Cisco 2960 Switch — SW1
- 2 × Admin PCs
- 2 × Sales PCs
- 2 × IT PCs
- 1 × Payroll Server representing a sensitive cloud resource

### Connections

- R1 GigabitEthernet0/1 ↔ SW1 GigabitEthernet0/1 — 802.1Q trunk
- Admin PCs → SW1 Fa0/1–Fa0/2
- Sales PCs → SW1 Fa0/3–Fa0/4
- IT PCs → SW1 Fa0/5–Fa0/6
- Payroll Server → SW1 Fa0/7

### Network Design

                    R1
             Cisco 2911 Router
                    |
             802.1Q Trunk
                    |
                   SW1
             Cisco 2960 Switch
                    |
       +------------+------------+------------+
       |            |            |            |
    VLAN 100      VLAN 10      VLAN 20      VLAN 30
    PAYROLL       ADMIN         SALES         IT
       |            |            |            |
    SERVER         2 PCs       2 PCs        2 PCs

The router uses router-on-a-stick subinterfaces to provide the default gateway for each VLAN and route traffic between the segmented networks.

## VLAN Segmentation

VLANs were used to separate departments and the sensitive Payroll resource into distinct Layer 2 broadcast domains.

| VLAN | Name | Network | Gateway |
|---|---|---|---|
| 10 | ADMIN | 192.168.10.0/24 | 192.168.10.1 |
| 20 | SALES | 192.168.20.0/24 | 192.168.20.1 |
| 30 | IT | 192.168.30.0/24 | 192.168.30.1 |
| 100 | PAYROLL | 192.168.100.0/24 | 192.168.100.1 |

The Payroll VLAN is separated from the departmental networks to represent a sensitive resource that should not be directly accessible by every department.

Access ports were assigned to the appropriate VLAN, while the router connection uses an 802.1Q trunk to carry the VLAN traffic between the switch and router.

## Inter-VLAN Routing

Inter-VLAN routing was implemented using a router-on-a-stick configuration on R1.

The physical connection between R1 and SW1 operates as an 802.1Q trunk. Separate router subinterfaces provide the default gateway for each VLAN:

| Subinterface | VLAN | IP Address |
|---|---|---|
| Gi0/1.10 | 10 | 192.168.10.1 |
| Gi0/1.20 | 20 | 192.168.20.1 |
| Gi0/1.30 | 30 | 192.168.30.1 |
| Gi0/1.100 | 100 | 192.168.100.1 |

This configuration allows traffic to be routed between the segmented networks while providing a central Layer 3 point where security controls can be applied.

Inter-VLAN connectivity was tested before implementing the ACL to establish a baseline and demonstrate the security risk of unrestricted communication.

## Security Risk

Before the ACL was implemented, inter-VLAN routing allowed unrestricted communication between the departmental networks and the Payroll resource.

For example, baseline testing demonstrated that a Sales workstation could successfully reach:

- Admin — `192.168.10.11`
- IT — `192.168.30.11`
- Payroll — `192.168.100.10`

Although inter-VLAN routing provided the required connectivity, unrestricted access created an unnecessary exposure to sensitive resources.

The security objective was therefore to preserve required business communication while restricting access based on the source department and destination network.

## ACL Security Policy

An extended ACL named `SALES_RESTRICTIONS` was configured on R1 to control traffic originating from the Sales VLAN.

The ACL was applied inbound on `Gi0/1.20`, the router subinterface for VLAN 20 (Sales).

|Source| Destination| Action|
|----|----|----|
|Sales| Admin| DENY|
|Sales| Payroll| DENY|
|Sales| IT| ALLOW|
|Sales| Other destinations| ALLOW|

The ACL only filters traffic originating from the Sales VLAN, so Admin and IT traffic is not affected by this specific ACL.

The ACL uses explicit deny rules for restricted destinations followed by permit rules for required Sales traffic. This ensures that Sales access to the sensitive Admin and Payroll networks is restricted while required Sales-to-IT communication remains available.

## Security Validation

Security controls were validated using controlled connectivity tests from the different departmental networks.

### Baseline Testing

Before the ACL was implemented, Sales was able to reach Admin, IT, and Payroll.

This established the initial security exposure and provided a baseline for comparison.

### Post-ACL Testing

After applying `SALES_RESTRICTIONS`:

| Test | Result |
|---|---|
| Sales → Admin | ❌ Blocked |
| Sales → Payroll | ❌ Blocked |
| Sales → IT | ✅ Allowed |
| Admin → Payroll | ✅ Allowed |
| IT → Payroll | ✅ Allowed |
| IT → Admin | ✅ Allowed |

ACL hit counters on R1 also confirmed that the deny and permit rules processed the corresponding traffic.

The results demonstrate that the ACL restricts unauthorized access to sensitive networks while maintaining required connectivity for authorized departments.

## Evidence

The following screenshots document the network configuration, security testing, and ACL enforcement performed during the lab.

|Evidence| Description|
|----|----|
|[01-vlan-segmentation.png](evidence/01-vlan-segmentation.png)| VLAN configuration and access-port assignments|
|[02-trunk-configuration.png](evidence/02-trunk-configuration.png)| 802.1Q trunk configuration between SW1 and R1|
|[03-inter-vlan-routing.png](evidence/03-inter-vlan-routing.png)| Router-on-a-stick subinterfaces and gateway status|
|[04-baseline-inter-vlan-connectivity.png](evidence/04-baseline-inter-vlan-connectivity.png)| Admin baseline connectivity across VLANs|
|[05-sales-baseline-access.png](evidence/05-sales-baseline-access.png)| Sales access before ACL enforcement|
|[06-acl-configuration.png](evidence/06-acl-configuration.png)| `SALES_RESTRICTIONS` ACL configuration|
|[07-acl-security-validation.png](evidence/07-acl-security-validation.png)| Sales access restrictions and permitted IT access|
|[08-admin-payroll-allowed.png](evidence/08-admin-payroll-allowed.png)| Authorized Admin access to Payroll|
|[09-it-access-validation.png](evidence/09-it-access-validation.png)| Authorized IT access validation|
|[10-acl-hit-counters.png](evidence/10-acl-hit-counters.png)| ACL rule match counters|
|[11-phase5-acl-enforcement.png](evidence/11-phase5-acl-enforcement.png)| Phase 5 ACL enforcement evidence|

## Lab Files

The completed Cisco Packet Tracer topology is available here:

[Download the Packet Tracer lab](packet-tracer/enterprise-network-segmentation-acl.pkt)

The file contains the configured VLANs, router-on-a-stick inter-VLAN routing, ACL security controls, end devices, and Payroll server used throughout the lab.

## Lessons Learned

This lab reinforced several practical network security concepts:

- VLANs provide logical segmentation between departments and sensitive resources.
- Inter-VLAN routing enables communication between segmented networks but can also introduce security exposure when unrestricted.
- Extended ACLs can enforce access policies based on source and destination networks.
- ACL placement is important because applying the ACL close to the source allows unwanted traffic to be filtered before it travels further into the network.
- Baseline testing helps identify security exposure before controls are implemented.
- Post-implementation testing confirms that security controls restrict unauthorized access without disrupting required business communication.
- ACL hit counters provide additional evidence that configured rules are actively processing traffic.
- Network segmentation and least-privilege access work together to reduce unnecessary access to sensitive resources.

## Conclusion

This lab demonstrated how enterprise network segmentation and access control can be used to protect sensitive resources.

VLANs separated the Admin, Sales, IT, and Payroll networks, while router-on-a-stick provided controlled inter-VLAN routing. Baseline testing showed that unrestricted routing allowed Sales to reach sensitive networks.

The `SALES_RESTRICTIONS` extended ACL was then implemented to block Sales access to Admin and Payroll while preserving required access to IT. Additional testing confirmed that authorized Admin and IT users could still reach the Payroll resource.

The combination of segmentation, controlled routing, ACL enforcement, and validation provides a practical example of applying least-privilege principles at the network layer.

