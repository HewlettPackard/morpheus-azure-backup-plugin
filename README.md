# Morpheus Azure Backup Plugin

The Morpheus Azure Backup Plugin integrates Morpheus with Azure Backup (Azure Recovery Services) to enable backup and restore of Azure virtual machines from within the Morpheus UI. It uses the Azure REST API (Azure Resource Manager) via HTTPS.

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Repository structure](#repository-structure)
- [Building the plugin](#building-the-plugin)
- [License](#license)
- [Installing](#installing)
- [Detailed Usage Steps](#detailed-usage-steps)
- [API Endpoints](#api-endpoints)

---

## Features

### Azure VM Backup and Restore

Back up and restore Azure virtual machines using Azure Recovery Services vaults and Azure Backup policies. Supports:

- Backup policy assignment per VM
- On-demand backup triggers
- Restore to the original VM
- Restore to a new VM (with a temporary staging storage account)
- Recovery point synchronisation

### Cloud Sync

Morpheus synchronises the following Azure Backup resources for inventory:

- Recovery Services vaults
- Backup policies (IaaSVM type)
- Protected items (VM backup jobs)
- Recovery points

---

## Requirements

| Requirement | Version |
|-------------|---------|
| Morpheus | 9.0.0 or later |
| Java | 25 or later |
| Gradle | Use the included Gradle wrapper (`./gradlew`) |

Additional prerequisites:

- An existing Azure cloud integration configured in Morpheus (the backup provider is scoped to an Azure cloud)
- An Azure subscription with at least one Recovery Services vault
- An Azure service principal or managed identity with the **Backup Contributor** role (or equivalent) on the target subscription or resource group
- Network access from the Morpheus appliance to `management.azure.com` and `login.microsoftonline.com` over HTTPS (port 443)

---

## Repository structure

```
src/main/groovy/com/morpheusdata/azure/
├── AzureBackupPlugin.groovy          - Plugin entry point; registers providers
├── AzureBackupProvider.groovy        - AbstractBackupProvider implementation; integration-level OptionTypes and lifecycle
├── AzureBackupTypeProvider.groovy    - BackupTypeProvider for VM-level backup config; resource group, vault, and storage account OptionTypes
├── datasets/
│   ├── VaultDatasetProvider.groovy   - Dataset provider supplying available vaults for UI selection
│   └── DatastoreDatasetProvider.groovy - Dataset provider supplying storage accounts for restore-to-new
├── services/
│   └── ApiService.groovy             - Azure REST API client (auth, vaults, policies, protected items, jobs, restore)
├── sync/
│   ├── VaultSync.groovy              - Syncs Recovery Services vaults
│   ├── PolicySync.groovy             - Syncs backup policies
│   └── RecoveryPointSync.groovy      - Syncs recovery points for protected items
└── util/                             - Shared utility classes (URL helpers, etc.)
src/assets/                           - Plugin icons (Azure-Backup-Center.svg)
src/test/groovy/                       - Spock unit tests
build.gradle, gradle.properties        - Build configuration and plugin metadata
```

---

## Building the plugin

Run the following command to compile and package the plugin jar:

```bash
./gradlew clean build
```

The packaged jar will be written to `build/libs/`.

To execute tests, use the following command:

```bash
./gradlew test
```

---

## License

This project is licensed under the Apache License 2.0.

See the [LICENSE](LICENSE) file for details.

---

## Installing

1. Build the plugin (see [Building the plugin](#building-the-plugin)) or download a released jar.
2. In Morpheus, navigate to **Administration > Integrations > Plugins**.
3. Click **Add** and upload the `morpheus-azure-backup-plugin-<version>.jar` from `build/libs/`.
4. Navigate to **Backups > Integrations > Add** and select **Azure** to configure the backup integration.

---

## Detailed Usage Steps

### Adding an Azure Backup Integration

1. Go to **Backups > Integrations > Add**.
2. Select **Azure** as the backup provider type.
3. Choose the **Cloud** (the Morpheus Azure cloud integration that holds the credentials).
4. Save. Morpheus connects to Azure and begins syncing Recovery Services vaults and policies.

### Assigning a Backup Policy to a VM

1. Go to **Provisioning > Instances**, open an Azure VM instance.
2. Click **Actions > Add Backup** or navigate to the **Backups** tab.
3. Select the **Resource Group** and **Vault** (populated from the synced Azure data).
4. The backup policy is applied at the vault level in Azure. Save to register the VM as a protected item.

### Running an On-Demand Backup

1. From the instance **Backups** tab, select a configured backup entry.
2. Click **Backup Now**. Morpheus triggers an on-demand backup job via the Azure Backup API.
3. Monitor job status in the **Backups** tab or in the Azure portal.

### Verifying Recovery Point Synchronisation

1. From the instance **Backups** tab, view the **Recovery Points** list.
2. Morpheus periodically syncs recovery points from Azure; click **Refresh** to trigger an immediate sync.

### Restoring to the Original VM

1. From the instance **Backups** tab, select a recovery point.
2. Click **Restore** and choose **Restore to existing**.
3. Select a **Storage Account** (used as a temporary staging location by Azure during restore).
4. Confirm. Morpheus calls the Azure Restore API and monitors the job to completion.

### Restoring to a New VM

1. From the instance **Backups** tab, select a recovery point.
2. Click **Restore** and choose **Restore to new**.
3. Select a **Storage Account** (used as a temporary staging location).
4. Confirm. Azure creates a new VM from the recovery point.

### Monitoring Backup and Restore Jobs

1. Backup job status is surfaced in the instance **Backups** tab and Morpheus activity log.
2. Job details, including status and error messages, are retrieved directly from the Azure Backup jobs API.

---

## API Endpoints

This plugin communicates with the **Azure Resource Manager REST API** (`https://management.azure.com`) and the **Azure AD token endpoint** (`https://login.microsoftonline.com`). All calls use HTTPS (port 443).

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `{identityUrl}/{tenantId}/oauth2/token` | POST | Acquire OAuth 2.0 bearer token |
| `/subscriptions` | GET | List available subscriptions |
| `/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.RecoveryServices/vaults` | GET | List Recovery Services vaults |
| `{vault}/backupPolicies?$filter=backupManagementType eq 'AzureIaasVM'` | GET | List backup policies for a vault |
| `{vault}/backupFabrics/Azure/refreshContainers` | POST | Trigger container discovery |
| `{vault}/backupProtectableItems?$filter=backupManagementType eq 'AzureIaasVM'` | GET | List discoverable VMs |
| `{vault}/backupProtectedItems` | GET | List protected (backed-up) items |
| `{vault}/backupFabrics/Azure/protectionContainers/{container}/protectedItems/{item}` | GET | Get protected item details |
| `{vault}/backupFabrics/Azure/protectionContainers/{container}/protectedItems/{item}` | PUT | Enable or update VM protection |
| `{vault}/backupFabrics/Azure/protectionContainers/{container}/protectedItems/{item}` | DELETE | Remove VM protection |
| `{vault}/backupFabrics/Azure/protectionContainers/{container}/protectedItems/{item}/backup` | POST | Trigger on-demand backup |
| `{vault}/backupFabrics/Azure/protectionContainers/{container}/protectedItems/{item}/recoveryPoints` | GET | List recovery points |
| `{vault}/backupFabrics/Azure/protectionContainers/{container}/protectedItems/{item}/recoveryPoints/{rp}/restore` | POST | Trigger restore |
| `{vault}/backupjobs` | GET | List backup jobs |
| `{vault}/backupjobs/{jobId}` | GET | Get job status |
| `{vault}/backupjobs/{jobId}/cancel` | POST | Cancel a running job |
| `/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Compute/virtualMachines/{vmId}` | GET | Get VM details for restore mapping |
