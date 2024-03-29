# StorageGRID Provisioning Process and Automation Testing

## Provisioning on StorageGRID has several components

Category            | Description
--------------------|----------------
Tenant              | Resource Container
User Mgmt           | Local & Federated Administrator(s) & Users
Key Mgmt            | User credentials (S3 Keys)
Certificate Mgmt    | Signed Certificate Mgmt
Bucket Mgmt         | Storage Resource

### Tenant Mgmt

Creating the resource container that will manage and present the object resources is the first step if a tenant doesn't exist or it isn't appropriate to use an existing tenant for any reason.

`netapp.storagegrid.na_sg_grid_account` - Create, Update, Delete Tenant Accounts on NetApp StorageGRID.  
`sg_tenant_mgmt.yml` - This creates a tenant based on a uniqueID. It will accept several other parameters to the initial creation or modification of the tenant proper.

* Name == Tenant name
* account_id == ID of the tenant and needed for managing the tenant and tenant resources
* Password - Set the password for the local administrative account, user 'root'
* quota_size - Total space the tenant can consume within the GRID. Defaults to `0` which is unlimited
* quota_size_unit - Defaults to `gb`
* Protocol == s3 - Commonly s3, however swift is also a supported option
* management - Enables/Disables the ability to manage the tenant through the "Tenant Manager". Useful for API only use cases
* 

This workflow can create, Modify or delete a tenant

Use Cases:
* Required, there will always be a tenant - A container for all resources presented externally
* Dedicated Tenant - Application, Department or end-user requiring power-user or more control for access keys, user & group access
* Shared Tenant - General object services where no user & basic key mgmt is acceptable

### User Mgmt

User management is a very important part of provisioning as it identifies who can administer the tenant and who can access tenant resources. This can be local accounts in small to medium deployments. However, this will likely be federated accounts & groups in the enterprise where security and centralized authentication is available.

Federated accounts & groups help manage account, group & passwords - Removing the need to manage those locally which is critical at enterprise scale.

Neither federated or local accounts replace the need for S3 protocol keys. These are still required for access object resources through a URI.

### Key Mgmt

S3 access keys are foundation for access control with the S3 object protocol. Being able to create/delete are fundamental to maintain control of who can access object resources within the 

### Certificate Mgmt

Signed certificates are a critical element of protocol security. Current & verified certificates help ensure endpoint identity. Without that, there is no guarantee the endpoint is what it says it is. Within the enterprise Signed certificates are broadly employed to implement security on approved resources.

### Bucket Mgmt

Buckets are the unit of storage that is presented to applications and users for use.

## Client Access Example using *AWS* CLI

Using StorageGRID with AWS S3 CLI
https://netapp.io/2016/12/13/using-storagegrid-webscale-aws-s3-cli/

Example Commands:
List buckets from known endpoint & valid tenant creds  
`aws s3 ls --profile default --endpoint-url https://192.168.7.217:10443/ --no-verify-ssl`

List contents of bucket - using downloaded CA bundle instead of OS  
`aws s3 ls s3://appid01-01 --profile default --endpoint-url https://192.168.7.217:10443 --ca-bundle appid01-certificate.pem`


Upload / Sync contents between locations
```
aws s3 sync vars/ s3://appid01-01 --profile default --endpoint-url https://192.168.7.217:10443 --ca-bundle appid01-certificate.pem
upload: vars/vars_sg_tenant_fabricpool.yml to s3://appid01-01/vars_sg_tenant_fabricpool.yml
upload: vars/vars_sg_config_exampleLab.yml to s3://appid01-01/vars_sg_config_exampleLab.yml
upload: vars/vars_sg_tenant_group_appid01.yml to s3://appid01-01/vars_sg_tenant_group_appid01.yml
upload: vars/vars_sg_tenant_users.yml to s3://appid01-01/vars_sg_tenant_users.yml
upload: vars/vars_sg_exampleLab.yml to s3://appid01-01/vars_sg_exampleLab.yml
upload: vars/vars_sg_tenant_buckets.yml to s3://appid01-01/vars_sg_tenant_buckets.yml
upload: vars/vars_sg_tenant_appid02.yml to s3://appid01-01/vars_sg_tenant_appid02.yml
upload: vars/vars_sg_tenant_creds.yml to s3://appid01-01/vars_sg_tenant_creds.yml
upload: vars/vars_sg_tenant_appid01.yml to s3://appid01-01/vars_sg_tenant_appid01.yml
upload: vars/vars_sg_tenant_users_appid01.yml to s3://appid01-01/vars_sg_tenant_users_appid01.yml
upload: vars/vars_sg_tenant_appid03.yml to s3://appid01-01/vars_sg_tenant_appid03.yml
upload: vars/sample_variables_file.yml to s3://appid01-01/sample_variables_file.yml
```

List bucket contents of a bucket

```
aws s3 ls s3://appid01-01 --profile default --endpoint-url https://192.168.7.217:10443 --ca-bundle appid01-certificate.pem
2022-12-06 09:29:05       1038 sample_variables_file.yml
2022-12-06 09:29:05        476 vars_sg_config_exampleLab.yml
2022-12-06 09:29:05        210 vars_sg_exampleLab.yml
2022-12-06 09:29:05       1909 vars_sg_tenant_appid01.yml
2022-12-06 09:29:05        601 vars_sg_tenant_appid02.yml
2022-12-06 09:29:05        394 vars_sg_tenant_appid03.yml
2022-12-06 09:29:05       1187 vars_sg_tenant_buckets.yml
2022-12-06 09:29:05        844 vars_sg_tenant_creds.yml
2022-12-06 09:29:05        421 vars_sg_tenant_fabricpool.yml
2022-12-06 09:29:05       1415 vars_sg_tenant_group_appid01.yml
2022-12-06 09:29:05        721 vars_sg_tenant_users.yml
2022-12-06 09:29:05        733 vars_sg_tenant_users_appid01.yml
```



