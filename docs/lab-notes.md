# Lab Notes — Network Connectivity Center Hub and Spoke

## Lab Goal

Build and validate a hub-and-spoke design with Google Cloud Network Connectivity Center using two VPC networks.

## Environment

The training environment started with:

- `spoke1-vpc`
  - `spoke1-subnet`
  - `10.10.30.0/24`
- `spoke2-vpc`
  - `spoke2-subnet`
  - `10.10.40.0/24`
- Lab-provided firewall rules for ICMP and remote administration

## Work Performed

### 1. Reviewed the existing VPCs and subnets

I confirmed that the two spoke networks were separate custom VPCs with different subnet ranges.

### 2. Reviewed the firewall rules

The lab already allowed ICMP, which meant I could use ping to test connectivity. The firewall rules did not create connectivity between the VPCs by themselves.

### 3. Created one VM in each VPC

- `spoke1-vm` received internal IP `10.10.30.2`
- `spoke2-vm` received internal IP `10.10.40.2`

### 4. Tested the baseline

Before NCC was configured:

- `spoke1-vm` to `10.10.40.2`: 100% packet loss
- `spoke2-vm` to `10.10.30.2`: 100% packet loss

This showed that the two VPCs did not have private IP connectivity at the start of the lab.

### 5. Created the NCC design

I created:

- Hub: `my-hub`
- Hub policy: Mesh topology
- VPC spoke: `spoke1` associated with `spoke1-vpc`
- VPC spoke: `spoke2` associated with `spoke2-vpc`

Both spokes became active.

### 6. Retested connectivity

After the NCC hub and spokes were active:

- `spoke1-vm` to `10.10.40.2`: 3/3 replies, 0% packet loss
- `spoke2-vm` to `10.10.30.2`: 3/3 replies, 0% packet loss

The same endpoints that failed before NCC could now communicate.

### 7. Reviewed Network Topology

I used Network Topology to view the VPC entities and traffic metrics in the environment.

## What I Learned

The clearest lesson from this lab was the difference between allowing traffic and having a route for that traffic. The ICMP firewall rules existed before NCC, but the two VPCs still could not reach each other. Once the VPCs were attached to the NCC hub as spokes, the same private IP tests succeeded.

I also got a better feel for why a centralized hub-and-spoke model can be easier to manage than building a larger number of individual pairwise connections.

## Why It Mattered

The lab gave me a simple way to validate a network architecture change using the same endpoints before and after the change. I could show that connectivity failed first, make the NCC change, and then prove that private IP communication worked afterward.

## Production Considerations

This was a guided training environment, so I would not copy every setting directly into production. In particular:

- Administrative source ranges should be restricted rather than broadly exposed.
- Firewall policy should follow least privilege.
- A production design would need to account for environment boundaries, routing policy, logging, monitoring, availability, ownership, and change control.
- The lab used a single temporary project and a small number of VPCs, so it did not test a larger multi-project or multi-organization design.
