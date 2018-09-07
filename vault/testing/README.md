# Creating a Self-Signed SSL Certificate
When using the SSL Endpoint feature for non-production applications, you can avoid the costs associated with the SSL certificate by using a self-signed SSL certificate. 
Though the certificate implements full encryption, visitors to your site will see a browser warning indicating that the certificate should not be trusted.
## Prerequisites
- Make sure you have openSSL installed: `which openssl`
- Otherwise `brew install openssl`
## Generate private key and certificate signing request
A private key and certificate signing request are required to create an SSL certificate. These can be generated with a few simple commands.

When the openssl req command asks for a “challenge password”, just press return, leaving the password empty. 
This password is used by Certificate Authorities to authenticate the certificate owner when they want to revoke their certificate. 
Since this is a self-signed certificate, there’s no way to revoke it via CRL (Certificate Revocation List).
- `openssl genrsa -des3 -passout pass:x -out server.pass.key 2048`
- `openssl rsa -passin pass:x -in server.pass.key -out server.key`
- `rm server.pass.key`
- `openssl req -new -key server.key -out server.csr`
## Generate SSL certificate
- `openssl x509 -req -sha256 -days 365 -in server.csr -signkey server.key -out server.crt`

# Test agent
- Change the version of the agent in the `conf.yaml` file
- Run `docker-compose -f docker-compose.yaml up -d`
- SSH to your container `docker exec -it scenario_1_dd-agent_1 /bin/bash`
- Run `/opt/datadog-agent/bin/agent/agent status` (in container)
  - Should see vault in the list aof checks run
- Run `/opt/datadog-agent/bin/agent/agent check process` and make sure we submit metrics.

NOTE: I am currently not able to use the certificate and the `ssl_verify` and `ssl_ignore_warning` options :\