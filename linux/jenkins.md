**Reference**
```
http://sam.gleske.net/blog/engineering/2016/05/04/jenkins-with-ssl.html <br>
https://certs.godaddy.com/repository <br>
```

 **Newer Basic Jenkins Install and Config**
 ```
# Install repo
wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key

# Install jenkins and java requirements
yum -y install java-1.8.0-openjdk java-1.8.0-openjdk-devel
yum -y install jenkins

# Create configuration
# - Change password and listen address
cat << EOF > /etc/sysconfig/jenkins
JENKINS_HOME="/var/lib/jenkins"
JENKINS_JAVA_CMD=""
JENKINS_USER="jenkins"
JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true"
JENKINS_PORT="-1"
JENKINS_LISTEN_ADDRESS=""
JENKINS_HTTPS_PORT="8443"
JENKINS_HTTPS_KEYSTORE="/var/lib/jenkins/certificates/jenkins.jks"
JENKINS_HTTPS_KEYSTORE_PASSWORD="password"
JENKINS_HTTPS_LISTEN_ADDRESS="EXT IP ADDY"
JENKINS_DEBUG_LEVEL="5"
JENKINS_ENABLE_ACCESS_LOG="no"
JENKINS_HANDLER_MAX="100"
JENKINS_HANDLER_IDLE="20"
JENKINS_ARGS=""
EOF

# Create directory, create keystore, fix permissions
# - Change OU
# - Change password
# - Document password in password manager
#
mkdir -p /var/lib/jenkins/certificates
keytool -genkey -alias replserver \
    -keyalg RSA -keystore /var/lib/jenkins/certificates/jenkins.jks \
    -dname "CN=localhost,OU=Dev, O=Dev, L=Dev, S=Dev, C=US" \
    -storepass password -keypass password
chown -R jenkins:jenkins /var/log/jenkins /var/lib/jenkins

#
# Certificiate
# - Grab the splat/star certificate and private key for the domain and drop it in /root/
# - Grab the godaddy root bundle and drop it in /root/
# - Run this command with the correct alias and filename
#
openssl pkcs12 -export -out jenkins_keystore.p12 \
-passout 'pass:password' -inkey star.key \
-in star.crt -certfile  godaddy-bundle.crt -name dev.local
  
keytool -importkeystore -srckeystore jenkins_keystore.p12 \
-srcstorepass 'password' -srcstoretype PKCS12 \
-srcalias dev.local -deststoretype JKS \
-destkeystore /var/lib/jenkins/certificates/jenkins.jks -deststorepass 'password' \
-destalias dev.local

#
# Start jenkins, grab password, create admin account
# 
systemctl start jenkins
cat /var/lib/jenkins/secrets/initialAdminPassword

# 
# Add plugins and restart
# - - Active Directory Plugin

#
# Configure AD plugin
# - Go to 'manage jenkins'
# - 'Configure global security'
# - Click 'enable security'
# - Click 'active directory'
# - Click 'add domain'
# - - Domain name: dev.local
# - - Domain Controller: ad01:3268
# - - CN=svc_acct_jenkins,OU=svc_accts,OU=Users,DC=dev,DC=net
# - - *use a previously created service account, password in password manager
# - Click 'Agents/TCP Port for JNLP/Fixed: 8088
# - Click Save
# - Log out and log in to test
```

**Basic Jenkins Installation**
```
# Getting repos
 wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
 rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key

# Installing Jenkins from the aforementioned repo
 yum -y install java-1.7.0-openjdk java-1.7.0-openjdk-devel
 yum -y install jenkins

# Make the Jenkins certificates directory:
 mkdir -pv /var/lib/jenkins/certificates/
 
# Generate the p12 file (using cert files from existing star chain - most likely can get these from load balancer in same geo)
 openssl pkcs12 -export -out certificate.p12 -inkey star.key -in star.crt -certfile chain.cer -name "mycert"
 
# Copy the jenkins.jks keystore file you created to the Jenkins certificates directory:
 cp -v jenkins.jks /var/lib/jenkins/certificates/jenkins.jks
 
# Import the p12 file into the main java keystore
keytool -importkeystore -deststorepass password -destkeystore jenkins.jks -srckeystore certificate.p12 -srcstoretype PKCS12
 
# Update the Jenkins configuration to disable HTTP and enable HTTPS solely:
 sed -i 's/^JENKINS_PORT="8080"/JENKINS_PORT="-1"/g' /etc/sysconfig/jenkins
 sed -i 's/^JENKINS_HTTPS_PORT=""/JENKINS_HTTPS_PORT="8443"/g' /etc/sysconfig/jenkins
 sed -i 's/^JENKINS_HTTPS_KEYSTORE=""/JENKINS_HTTPS_KEYSTORE="\/var\/lib\/jenkins\/certificates\/jenkins.jks"/g' /etc/sysconfig/jenkins
 sed -i 's/^JENKINS_HTTPS_KEYSTORE_PASSWORD=""/JENKINS_HTTPS_KEYSTORE_PASSWORD="password"/g' /etc/sysconfig/jenkins
 
# Restart the Jenkins service:
 systemctl restart jenkins 
 systemctl status jenkins
 ```
