# StorageGRID Provisioning Process and Automation Testing

## Provisioning on StorageGRID has several components

Category            | Decription
--------------------|----------------
Tenant              | Resource Contianer
User Mgmt           | Local & Federated Administrator(s) & Users
Key Mgmt            | User crednetials (S3 Keys)
Certificate Mgmt    | Signed Certificate Mgmt
Bucket Mgmt         | Storage Resource

### Tenant Mgmt

Creating the resource container that will manage and present the object resources is the first step if a tenant doesn't exist or it isn't appropreiate to use an existing tenant for any reason.

`netapp.storagegrid.na_sg_grid_account` - Create, Update, Delete Tenant Accounts on NetApp StorageGRID.  
`sg_tenant_mgmt.yml` - This creates a tenant based on a uniqueID. It will accept several other paramters to the initial creation or modification of the tenant proper.

* Name == Tenant name
* account_id == ID of the tenant and needed for managing the tenant and tenant resources
* Password - Set the password for the local administrative account, user 'root'
* quota_size - Total space the tenant can consume within the GRID. Defaults to `0` which is unlimited
* quota_size_unit - Defaults to `gb`
* Protocol == s3 - Commonly s3, however swift is also a supported option
* management - Enables/Disables the ability to manage the tenant through the "Tenant Manager". Useful for API only use cases
* 

### User Mgmt

User management is a very important part of provisioning as it identifies who can administer the tenant and who can access tenant resources. This can be local accounts in small to medium deployments. However, this will likely be federated accounts & groups in the enterprise where security and centralized authentication is available.

Federated accounts & groups help manage account, group & passwords - Removing the need to manage those locally which is critical at enterprise scale.

Neither federated or local accounts replace the need for S3 protocol keys. These are still required for access object resources through a URI.

### Key Mgmt

S3 access keys are foundation for access control with the S3 object protocol. Being able to create/delete are fundemental to maintin control of who can access objehct resources within the 

### Certificate Mgmt

Signed certificates are a critical element of protocol security. Current & verified certificates help ensure endpoint identity. Without that, there is no guarentee the endpoint is what it says it is. Within the enterprise Signed certificates are broadly employed to implement security on approved resources.

### Bucket Mgmt

Buckets are the unit of storage that is presented to applications and users for use.

