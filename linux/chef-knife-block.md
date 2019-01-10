## knife block installation
```
(install chef-dk)
yum install ruby rubygem
gem install knife-block
```
## adding a chef server to admin
```
cat << EOF > .chef/admin-server01.pem
(private key)
EOF

cat << EOF > .chef/knife-server01.rb
log_level                :info
log_location             STDOUT
node_name                'admin'
client_key               '/root/.chef/admin-server01.pem'
chef_server_url          'https://server01/organizations/acme'
syntax_check_cache_path  '/root/.chef/syntax_check_cache'
EOF

knife block list
knife block use server01
```
