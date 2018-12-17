**Add user and password on chef-server**
```
chef-server-ctl user-create (username) (firstname) (lastname) (email) (somepassword)
chef-server-ctl org-user-add wfs (username)
```

**Configure knife under your user**
```
knife configure initial
```

**knife configuration details**
```
 log_level                :info
 log_location             STDOUT
 node_name                '(username)'
 client_key               '/home/dev.local/(username)/.chef/(username).pem'
 validation_client_name   'dev-validator'
 validation_key           '/etc/opscode/dev-validator.pem'
 chef_server_url          'https://dev.local/organizations/dev'
 syntax_check_cache_path  '/home/dev.local/(username)/.chef/syntax_check_cache'
```
**Self signed certificates will error, workaround**
```
knife ssl fetch
```
