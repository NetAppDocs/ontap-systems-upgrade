## Copilot instructions for ONTAP controller hardware upgrade documentation

### Repository overview
Product: ONTAP controller hardware upgrades

This repository documents how to upgrade NetApp *AFF*, *ASA*, and *FAS* controller hardware in ONTAP environments. The content helps users choose and execute the correct upgrade method by *aggregate relocation (ARL)*, *moving volumes*, *moving storage*, or supported *drive shelf* conversion workflows.
The procedure-selection entry point is `choose_controller_upgrade_procedure.adoc`.

**Page title guidance:**
When writing or revising page titles for this repository, do not use the product name literally as a prefix. Titles should describe the topic in context, not label it with the product name.

Use "ONTAP controller hardware" as the default contextual anchor for general-scope page titles. When a title lists platform families only as a broad qualifier, replace the family list with "your ONTAP controller hardware". For example, use "Choose an upgrade procedure for your ONTAP controller hardware" rather than "Choose an upgrade procedure for AFF, ASA, or FAS controller hardware".

Retain specific model numbers when the model is the functional differentiator of the procedure and removing it would make the title ambiguous or incorrect. This includes model-specific in-chassis controller replacement procedures and model-specific drive shelf conversion procedures. Do not generalize those titles to "ONTAP controller hardware".

Decision rule:

- If the page applies broadly across families, use "ONTAP controller hardware".
- If the page applies only to named models, keep the model names in the title.

### Repository structure

- `_include/` – Shared AsciiDoc fragments reused across procedures, including `ru_all_*`, `ru_auto_*`, `ru_man_*`, and `ru_upgrade_*` naming patterns; newer files might use hyphens instead of underscores (for example, `ru-auto-*`).
- `media/` – Diagrams and hardware images for node replacement, chassis work, cabling, disk reassignment, and workflow illustrations.
- `redirects/` – Redirect and legacy-named topic files that preserve older upgrade page paths and topic names.
- `upgrade/` – Controller upgrade procedures based on moving volumes, moving storage, and supported convert-to-drive-shelf workflows.
- `upgrade-arl/` – ARL entry content that helps readers choose the correct aggregate-relocation procedure and supported upgrade path.
- `upgrade-arl-auto/` – ARL procedures that use `system controller replace` commands for systems running **ONTAP 9.5, 9.6, or 9.7**. Includes MetroCluster verification steps.
- `upgrade-arl-auto-app/` – ARL procedures that use `system controller replace` commands with automated port reachability checks, for systems running **ONTAP 9.8 or later**.
- `upgrade-arl-auto-app-9151/` – ARL procedures using `system controller replace` commands for AFF and FAS controllers introduced in **ONTAP 9.15.1 or later**. MetroCluster FC and IP configurations are **not supported** by this procedure.
- `upgrade-arl-auto-in-chassis/` – ARL procedures that convert specific supported systems (such as AFF A700→AFF A900 or AFF A800) to replacement controllers while keeping the existing chassis and disks.
- `upgrade-arl-manual/` – Manual ARL procedures for systems running **ONTAP 9.7 or earlier**. Includes switch reconfiguration steps (RCF files, recabling) for certain platforms.
- `upgrade-arl-manual-app/` – Manual ARL procedures for systems running **ONTAP 9.8 or later**.

### Product-specific context
**Architecture and components:**
- Upgrades are performed on ONTAP *HA pairs* inside a cluster; larger clusters are upgraded one HA pair at a time.
- *ARL* relocates ownership of *non-root aggregates* between nodes that share storage in the same cluster so data stays available during controller replacement.
- In the docs, the original controllers are *node1* and *node2*, and the replacement controllers are *node3* and *node4*; after the upgrade, the replacement nodes take over the original node identities.
- Some workflows replace controllers while keeping the same *chassis* and disks, and others convert a supported original system into a *drive shelf* attached to the replacement nodes.
- *MetroCluster* is a special topology with procedure-specific support: `upgrade-arl-auto/`, `upgrade-arl-manual/`, and `upgrade-arl-manual-app/` include MetroCluster verification steps and support MetroCluster configurations; `upgrade-arl-auto-app-9151/` explicitly excludes MetroCluster FC and IP. Do not assume every upgrade path supports MetroCluster.

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
**Choose an upgrade procedure:** Start from `choose_controller_upgrade_procedure.adoc`, which contains the decision table → identify original and replacement platforms → determine the ONTAP version running on the original nodes → determine whether the upgrade must be disruptive or nondisruptive → determine whether the system is modular, integrated, in-chassis, or drive-shelf-convertible → follow the matching ARL, move-volumes, or move-storage procedure. Note: if the original and new nodes run different ONTAP versions, a software upgrade may be required first; version differences cannot exceed four releases.

**Upgrade by using ARL:** Confirm the supported controller replacement path → prepare node1/node2 and replacement node3/node4 → relocate non-root aggregates and migrate LIFs during controller replacement → verify the replacement nodes and resume related services

**Upgrade by moving volumes:** Prepare the original nodes → install and join the new nodes to the cluster → move volumes and required LIFs, including SAN-specific steps when present → remove the original nodes and complete post-upgrade configuration

**Upgrade by moving storage:** Gather system, license, and port details → shut down and remove the original nodes → install and configure the new nodes, then attach shelves and reassign disks → restore root volume configuration and complete final network and upgrade steps
