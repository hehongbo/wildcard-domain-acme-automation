# Wildcard Domain ACME Automation

Apply for *Let's Encrypt*'s wildcard certificates for second-level-domains with Drone CI, in both RSA and EC (Elliptic Curve) algorithms, and publish the public key for future uses. 

The idea is to apply for wildcard certificates and use them elsewhere, deploying them in other automation procedures, instead of applying for one keypair per server. This might also help to reduce exposures of what you've deployed on searching services like [crt.sh](https://crt.sh/).

Related infrastructures and services:
- **Drone CI** and Drone's Docker Runner for running signing procedures.
- **BitBucket**. It doesn't matter from where Drone CI clones the repository, but the script assumes that we're using BitBucket and publish obtained certificates to Bitbucket's *Download* panel with their API.
- **deSEC** for hosting the domain's DNS and providing API accesses for adding TXT records to respond to LE's ACME challenges.
- **Let's Encrypt** for providing certificates. In theory, it would work with other certificate providers with ACME protocol.

## Disclaimer

This automation is **HARD-CODED** with APIs of these infrastructures above, so it should be more like a demonstration instead of a template that you can fill in your own values and deploy right away. 

If you'd like to host your repository with other hosting services like GitHub, GitLab, run a similar procedure with public building services like GitHub Actions, Travis CI, or your domain is hosted with providers other than deSEC, you might have to modify the code slightly or make another one.  

Also, no exception handling in this automation. Running CI tasks with empty or wrong variables provided will results in build failures, or hitting the rate limit of Let's Encrypt.

## A more detailed introduction

This automation obtains certificates in RSA and EC algorithms that cover the following DNS names, assuming you're signing `example.com`:
- `example.com`
- `*.example.com`

That would satisfy the needs of deploying personal websites, as well as self-hosted services. 

After signing, this automation will append `yourdomain_YYYYMM_rsa.pem`, `yourdomain_YYYYMM_ec.pem` to the *Downloads* panel of the given BitBucket repository, and override `latest.json` in the *Downloads* that tells where you can download these certificates, from both `letsencrypt.org` and the repository itself. This might be useful when deploying other services with the obtained certificate, in an automated fashion.
  
This automation uses LEGO to respond to LE's ACME challenge and obtaining certificates.

## Variables to provide when using this automation

The following variables should be filled into `.drone.yml` before committing and triggering CI tasks:

| Variable              | Description                                                                                                                                                       |  
| :-------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ACME_ACCOUNT_MAIL`   | E-mail address to use as an account when using LE's services.                                                                                                     |
| `ACME_SERVER`         | LE's server URL. <br /> `https://acme-v02.api.letsencrypt.org/directory` for production and `https://acme-staging-v02.api.letsencrypt.org/directory` for testing. |
| `BITBUCKET_REPO_NAME` | Bitbucket repository to push to. <br />Ex. `someone/some-repo`                                                                                                    |
| `BITBUCKET_USERNAME`  | Bitbucket username. <br />Ex. `someone`                                                                                                                           |
| `DOMAIN`              | Domain to obtain certificates with.                                                                                                                               |
| `TIMEZONE`            | Timezone to reference when filling in the obtaining date of certificates. <br />Ex. `Asia/Shanghai`                                                               |

The following variables should be provided as *Secret*s before triggering CI tasks: *(Note that it's okay to provide multi-line PEM private keys with Drone CI's Secret feature)*

| Variable             | Description                                                                                                                                      |  
| :------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------- |  
| `ACME_ACCOUNT_KEY`   | The private key to authenticate LE's services. In RSA private key format. <br />Can be generated with: <br />`openssl genrsa 4096 > account.key` |    
| `BITBUCKET_PASSWORD` | Password to authenticate bitbucket account when pushing release downloads.                                                                       |    
| `DESEC_TOKEN`        | Token of deSEC DNS API.                                                                                                                          |   
| `DOMAIN_EC_KEY`      | Pre-generated EC private key. <br />Can be generated with: <br />`openssl genrsa 4096 > domain_rsa.key`                                          |  
| `DOMAIN_RSA_KEY`     | Pre-generated RSA private key. <br />Can be generated with: <br />`openssl ecparam -name prime256v1 -genkey -noout -out domain_ec.key`           |   
