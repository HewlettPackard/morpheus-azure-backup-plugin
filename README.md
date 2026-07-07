# Morpheus Azure Backup Plugin

This plugin provides backup integration between [Microsoft Azure Backup](https://azure.microsoft.com/en-us/products/backup) and [Morpheus](https://morpheusdata.com). It enables Recovery Services vault discovery, backup policy sync, Azure VM backup protection, recovery point sync, and VM restore workflows from within the Morpheus platform.

## Requirements

| Component | Minimum Version |
|-----------|----------------|
| Morpheus | 9.0.0 |

## Installation

1. Download the latest `.jar` from the [Releases](https://github.com/HewlettPackard/morpheus-azure-backup-plugin/releases) page, or [build it yourself](#building).
2. In Morpheus, navigate to **Administration → Integrations → Plugins**.
3. Click **Browse** and upload the `.jar` file.
4. The **Azure** backup integration will appear after the plugin loads.

## Configuration

When adding an Azure backup integration in Morpheus (**Backups → Integrations → Add Backup Integration**), provide the following:

| Field | Description |
|-------|-------------|
| **Cloud** | Existing Azure cloud integration used for Azure credentials and inventory context |
| **Resource Group** | Azure resource group containing the Recovery Services vault used for a backup |
| **Vault** | Recovery Services vault selected for the backup configuration |
| **Storage Account** | Azure storage account used as a temporary location during restore workflows |

## Features

### Backup Integration
The plugin registers an Azure `BackupProvider` that connects Morpheus backup workflows to Azure Backup. Supported integration behavior includes:

- Validate connectivity using the selected Azure cloud credentials
- Add Azure VM backups to existing Azure backup jobs
- Execute backup jobs through Morpheus
- Delete Azure backup policies when backup jobs are removed
- Track provider health during refresh

### Azure Backup Sync
The following Azure Backup resources are discovered and kept in sync:

- **Recovery Services Vaults** — vaults discovered from Azure resource groups
- **Backup Policies** — Azure backup policies represented as Morpheus backup jobs
- **Recovery Points** — Azure VM recovery points represented as Morpheus backup results

### Backup and Restore Operations
Azure VM protection and restore workflows are available through the Morpheus backup framework. Supported operations include:

- Enable Azure Backup protection for a VM using a selected resource group, vault, and policy
- Cache and match Azure protectable VMs before enabling protection
- Restore backups to the original VM location
- Restore backups to a new VM location using a selected storage account
- Poll Azure restore jobs and update Morpheus restore status

## Building

```bash
./gradlew shadowJar
```

The plugin JAR will be written to `build/libs/`.

## License

Copyright 2022 Morpheus Data, LLC. Licensed under the [Apache License, Version 2.0](LICENSE).
