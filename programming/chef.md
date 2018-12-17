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
