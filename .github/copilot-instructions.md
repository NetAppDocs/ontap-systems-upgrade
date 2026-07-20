## Copilot instructions for ONTAP controller hardware upgrade documentation

### Repository overview
Product: ONTAP controller hardware upgrades

This repository documents how to upgrade NetApp *AFF*, *ASA*, and *FAS* controller hardware in ONTAP environments. The content helps users choose and execute the correct upgrade method by *aggregate relocation (ARL)*, *moving volumes*, *moving storage*, or supported *drive shelf* conversion workflows.

### Repository structure
- `_include/` – Shared AsciiDoc fragments reused across ARL, move-volumes, move-storage, and troubleshooting topics.
- `media/` – Diagrams and hardware images for node replacement, chassis work, cabling, disk reassignment, and workflow illustrations.
- `redirects/` – Redirect and legacy-named topic files that preserve older upgrade page paths and topic names.
- `upgrade/` – Controller upgrade procedures based on moving volumes, moving storage, and supported convert-to-drive-shelf workflows.
- `upgrade-arl/` – ARL entry content that helps readers choose the correct aggregate-relocation procedure and supported upgrade path.
- `upgrade-arl-auto/` – ARL procedures that use `system controller replace` commands in the older automated command-based flow.
- `upgrade-arl-auto-app/` – ARL procedures that use `system controller replace` commands with automated port reachability checks.
- `upgrade-arl-auto-app-9151/` – ARL procedures for newer controller families that keep the existing disks and data in place.
- `upgrade-arl-auto-in-chassis/` – ARL procedures that keep the existing chassis and disks while replacing controllers or controller modules.
- `upgrade-arl-manual/` – Manual ARL procedures for earlier controller-upgrade paths.
- `upgrade-arl-manual-app/` – Manual ARL procedures for later same-family AFF, ASA, and FAS controller upgrades.

### Product-specific context
**Architecture and components:**
- Upgrades are performed on ONTAP *HA pairs* inside a cluster; larger clusters are upgraded one HA pair at a time.
- *ARL* relocates ownership of *non-root aggregates* between nodes that share storage in the same cluster so data stays available during controller replacement.
- In the docs, the original controllers are *node1* and *node2*, and the replacement controllers are *node3* and *node4*; after the upgrade, the replacement nodes take over the original node identities.
- Some workflows replace controllers while keeping the same *chassis* and disks, and others convert a supported original system into a *drive shelf* attached to the replacement nodes.
- *MetroCluster* is a special topology with separate guidance or procedure-specific limits; do not assume every upgrade path supports every MetroCluster configuration.

**Key concepts:**
- *Move volumes* is a nondisruptive method that adds new nodes to the cluster, moves volumes to them, migrates *LIFs*, and then removes the original nodes.
- *Move storage* is a disruptive method that shuts down the original nodes, installs replacement nodes, reassigns disks, restores root volume configuration, and remaps ports.
- *Integrated systems* have internal disks in the controller chassis; *modular systems* keep storage in externally attached shelves.
- *LIFs* include non-SAN data LIFs, SAN LIFs, and cluster-management LIFs; many procedures also require port mapping or interface-group reassignment on replacement hardware.
- *SVM* relocation is not part of the main controller-upgrade flows in this repo and is referenced as a separate migration path.

**Naming conventions and terminology:**
- Use the platform families exactly as written: *AFF*, *ASA*, and *FAS*.
- Keep series terminology exact where the docs do: *A-Series* and *C-Series* are distinct, and some manual upgrade paths keep replacements within the same series.
- *ARL* means *aggregate relocation*; *HA* means *high availability*; *LIF* means *logical interface*; *SP* means *Service Processor*.
- `system controller replace` is the exact ONTAP command-based workflow name used throughout the automated ARL content.
- Hardware terms such as *NVRAM*, *I/O modules*, *cluster ports*, *disk ownership*, and *root volume configuration* are central to these procedures and should be used precisely.

### Typical user workflows
**Choose an upgrade procedure:** Identify original and replacement platforms → determine whether the upgrade must be disruptive or nondisruptive → determine whether the system is modular, integrated, in-chassis, or drive-shelf-convertible → follow the matching ARL, move-volumes, or move-storage procedure

**Upgrade by using ARL:** Confirm the supported controller replacement path → prepare node1/node2 and replacement node3/node4 → relocate non-root aggregates and migrate LIFs during controller replacement → verify the replacement nodes and resume related services

**Upgrade by moving volumes:** Prepare the original nodes → install and join the new nodes to the cluster → move volumes and required LIFs, including SAN-specific steps when present → remove the original nodes and complete post-upgrade configuration

**Upgrade by moving storage:** Gather system, license, and port details → shut down and remove the original nodes → install and configure the new nodes, then attach shelves and reassign disks → restore root volume configuration and complete final network and upgrade steps
