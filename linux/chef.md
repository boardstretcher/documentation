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



**bootstrapping a node with sudo, chef-client and a pinned version**
```
$HOST="dev.local"; knife bootstrap $HOST --sudo -x boardstretcher -E my-environment -P --node-name $HOST -N $HOST -r recipe[chef-client::default] --bootstrap-version 12.21.3
```

**Using an encrypted databag**
```
# The recipe
if(File.file?("/etc/chef/#{node['auth']['bagname']}"))
  secret = Chef::EncryptedDataBagItem.load_secret("/etc/chef/#{node['auth']['bagname']}")
  enc_dbag = Chef::EncryptedDataBagItem.load("encrypted_environments", node.environment, secret)
	template '/etc/sssd/sssd.conf' do
	  source 'sssd.conf.erb'
	  variables sssd_domain: enc_dbag['auth']['domain'],
	  notifies :restart, 'service[sssd]'
	end
end

# The environment file
{
    "name": "datacenter1",
    "description": "my first datacenter",
    "chef_type": "environment",
    "json_class": "Chef::Environment",
    "default_attributes": {
        "auth" : {
            "bagname": "my-databag"
         }
    }
}

# The databag
{
  "id": "datacenter1",
  "auth": {
    "domain": "dev.local"
  }
}

# The template
[sssd]
config_file_version = 2
debug_level = 3
reconnection_retries = 3
sbus_timeout = 30
services = nss, pam
domains = <%= @sssd_domain %>

# The commands
# Create the databag key
openssl rand -base64 512 | tr -d '\r\n' > databag_key
# Create the databag
knife data bag create encrypted_environments datacenter1 --secret-file ./databag_key
# Edit the databag
knife data bag edit encrypted_environments datacenter1 --secret-file databag-key
```
