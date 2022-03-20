# ONTAP Certificate Authentication - SSL Certs

### Create SSL Cert for admin
`openssl req -x509 -nodes -days 1095 -newkey rsa:2048 -keyout admin.key -out admin.pem -subj “/C=US/ST=NC/L=RTP/O=NetApp/CN=admin”`

*Note: copy the contents of the .pem file:  including the —–BEGIN CERTIFICATE—— and —–END CERTIFICATE—– parts*

### Install client-ca SSL Cert for admin on ONTAP
`security certificate install -type client-ca -cert-name admin -vserver cluster1`

### Enable client-ca SSL Cert for admin
`security ssl modify -vserver cluster1 -client-enabled true`

### Enable SSL Cert auth method for admin
`security login create -user-or-group-name admin -application http -authentication-method cert`
`security login create -user-or-group-name admin -application ontapi -authentication-method cert`

## Ansible Setup & Test

#### Leveraging SSL authentication
* Original section example:
  ```
  hostname: Ansible01
  username: admin
  password: netapp123
  https: true
  validate_certs: false
  ```

*Note: With the .pem and .key files in the same directory as the playbook you could instead use this:*

* Updated section example:
  ```
  hostname: Ansible01
  validate_certs: false
  cert_filepath: admin.pem
  key_filepath: admin.key
  ```

#### SSL Cert Highlights
  * username not required
  * https not required
  * local or FQ-Path to filepath(s)


## curl & API Test
Test access to ONTAP cluster using certificate. Replace <ONTAP Management LIF> and <vserver name> with Management LIF IP and SVM name.

Note: Remeber to pass `-k` or `--insecure` to disable self-sign certificate checking

`curl -X POST -Lk https://<ONTAP-Management-LIF>/servlets/netapp.servlets.admin.XMLrequest_filer --key k8senv.key --cert ~/k8senv.pem -d '<?xml version="1.0" encoding="UTF-8"?><netapp xmlns="http://www.netapp.com/filer/admin" version="1.21" vfiler="<vserver-name>"><vserver-get></vserver-get></netapp>'`

```
curl -X POST -Lk https://192.168.7.200/servlets/netapp.servlets.admin.XMLrequest_filer --key admin.key --cert admin.pem -d '<?xml version="1.0" encoding="UTF-8"?><netapp xmlns="http://www.netapp.com/filer/admin" version="1.21" vfiler="ontap-vs01"><vserver-get></vserver-get></netapp>'
```

### Displays all SSL Certs on ONTAP Target

`curl -X GET --insecure "https://192.168.7.200/api/security/certificates?fields=common_name" -H "accept: application/hal+json"  --key admin.key --cert admin.pem`

### Curl command to get aggregates

`curl -kiu USERNAME:PASSWORD -X GET https://CLUSTERNAME/api/storage/aggregates`
`curl -kiu admin:netapp1234 -X GET https://192.168.7.200/api/storage/aggregates`
or
`curl -ki --key admin.key --cert admin.pem -X GET https://CLUSTERNAME/api/storage/aggregates`
`curl -ki --key admin.key --cert admin.pem -X GET https://192.168.7.200/api/storage/aggregates`
`curl -ki --key admin.key --cert admin.pem -X GET https://192.168.7.200/api/storage/aggregates?fields=name`

### Curl command to get version

`curl -ki --key admin.key --cert admin.pem -X GET https://192.168.7.200/api/cluster?fields=version`

### Curl command to get QOS policies

`curl -ki --key admin.key --cert admin.pem -X GET https://192.168.7.200/api/storage/qos/policies`+