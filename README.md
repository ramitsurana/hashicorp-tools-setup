# Vault
A Sample art of work on Vault

## Architecture Diagram

![layers-368ccce4](https://user-images.githubusercontent.com/8342133/34358547-ac3e3734-ea76-11e7-85dc-f1b65cc7d307.png)

## Setup

* [Installation](#installation)
* [Dev Mode](#dev-mode)
* [Mounts](#mounts)
* [Read/Write Token](#read/write-token)
* [SSH](#ssh)
* [Vault UI](#vault-ui)
* [Github Authentication](#github-authentication)
* [AWS Authentication](#aws-authentication)
* [Policies](#policies)
* [Transit](#transit)

#### Installation

Download the binary from https://www.vaultproject.io/downloads.html

````
$ chmod +x vault
$ mv vault /usr/local/bin/
$ export VAULT_ADDR='http://127.0.0.1:8200'
````

#### Dev mode

````
$ sudo vault server -dev
````

#### Mounts

````
//List mounts
$ sudo vault mounts
````

#### Read/Write Token

````
//Storing a password for a user
$ vault write secret/ramit value=SuperSecretPassword

//Storing a token with password for a sample user
$ vault write secret/ramit value=SuperSecretPassword token=ABCDE12345

//Read token
$ vault read secret/ramit

//Read in a particular format
# JSON Format
$ vault read --format=json secret/ramit
# YAML Format
$ vault read --format=yaml secret/ramit
# Table Format
$ $ vault read --format=table secret/ramit

//Store multiple values for a User using a file
$ vault write secret/password @data.json
````

#### SSH

````
//Mount ssh at vault
$ vault mount ssh
````

#### Vault UI

````
Via Docker
$ sudo docker run -d -p 8000:8000 --name vault-ui djenriquez/vault-ui
````

#### GitHub Authentication

````
//Enable Vault Github Auth
$ vault auth-enable github

//Vault Write GitHub Creds
$ vault write auth/github/config

//With Root
$ vault write auth/github/map/teams/default value=root

//Revoke all Github tokens
$ vault token-revoke -mode=path auth/github

//Disable Github from Vault
$ vault auth-disable github
````
#### AWS Authentication

````
Mount AWS on Vault
$ vault mount aws

//Vault Write AWS Creds
$ vault write aws/config/root  access_key=ABCDE1234  secret_key=1234ABCDE

//Using IAM Policy in JSON Format with policy.json
$ vault write aws/roles/deploy policy=@policy.json

//Read & Verify the AWS tokens in Vault
$ vault read aws/creds/deploy

//Using lease_id parameter from above in the above command
$ vault revoke <aws/creds/deploy/185e6910-6d36-e9a6-33b3-fc8dcfd4e97c> → lease_id
````

#### Policies

````
//List all Policies on Vault
$ vault policies

//Implementing (acl.hcl) on vault with name (secret)
$ vault policy-write secret acl.hcl

//Delete Policy in Vault
$ vault policy-delete secret

//Create a new token using a particular policy
$ vault token-create -policy="secret"
````

#### Transit

````
//Mount transit on vault
$ vault mount transit

//Create a named encryption key
$ vault write -f transit/keys/foo

//Validate the key foo
$ vault read transit/keys/foo

//Encrypt the Plain text (the quick brown fox)
echo -n "the quick brown fox" | base64 | vault write transit/encrypt/foo plaintext=-

Decrypt the Plain text 
$ vault write transit/decrypt/foo ciphertext=vault:v1:czEwyKqGZY/limnuzDCUUe5AK0tbBObWqeZgFqxCuIqq7A84SeiOq3sKD0Y/KUvv

$ echo "dGhlIHF1aWNrIGJyb3duIGZveAo=" | base64 -d
````

## License

MIT License

## References

[1] [hashi-vault-utils](https://github.com/Voronenko/hashi_vault_utils)








