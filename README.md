# Wildcard Domain ACME Automation

Apply for *Let's Encrypt*'s wildcard certificates for second-level-domains with Drone CI, in both RSA and EC (Elliptic Curve) algorithms.

The idea is to apply for wildcard certificates and use them elsewhere, deploying them in other automation procedures, instead of applying for one keypair per server. This might also help to reduce exposures of what you've deployed on searching services like [crt.sh](https://crt.sh/).

## Disclaimer

The `.drone.yml` that comes with this repository is not something you can commit and push and deploy right away. The DNS provider I use as an example is ClouDNS, you might want to refer to LEGO's manual, to figure out which variable to fill in when using a different DNS provider (as long as it's supported by LEGO).

Also, no exception handling in this automation. Running CI tasks with empty or wrong variables provided will results in build failures, or hitting the rate limit of Let's Encrypt.

## A more detailed introduction

This automation obtains certificates in RSA and EC algorithms that cover the following DNS names, assuming you're signing `example.com`:
- `example.com`
- `*.example.com`

That would satisfy the needs of deploying personal websites, as well as self-hosted services. 

This automation uses LEGO to respond to LE's ACME challenge and obtaining certificates. If the signing is successful, `rsa.cert.pem` and `ec.cert.pem`, which returned by LE's server, will be left within the workspace. You can append your own steps, commit or publish them somewhere else, and fetch them in other automation procedures.

## Instructions

1. Fill in the name of domain you own, in the `DOMAIN` variable at the beginning of `.drone.yml`.
2. Fill in the E-mail address you want to use as an account, in the `ACME_ACCOUNT_MAIL` variable.
3. If you're testing the automation, it's usually good idea to temporarily change the `ACME_SERVER` variable from `https://acme-v02.api.letsencrypt.org/directory` to `https://acme-staging-v02.api.letsencrypt.org/directory`.
4. Create a repository to hold this automation, trigger a sync in Drone CI if needed, and activate the repository in Drone CI.
5. Create a secret named `ACME_ACCOUNT_KEY` under the repository's secret panel, and fill in the private key of your Let's Encrypt account. *(Note that it's okay to provide multi-line PEM private keys with Drone CI's Secret feature)* \
   To generate an account private key, use `openssl genrsa 4096 > account.key`.
6. Create a secret named `DOMAIN_RSA_KEY`, and fill in the RSA private key which you want to apply certificates with. \
   To generate an RSA key for your domain, use `openssl genrsa 4096 > domain_rsa.key`.
7. Create a secret named `DOMAIN_EC_KEY`, and fill in the EC private key which you want to apply certificates with. \
   To generate an EC key for your domain, use `openssl ecparam -name prime256v1 -genkey -noout -out domain_ec.key`.
8. Go to LEGO project's [manual](https://go-acme.github.io/lego/dns/), find the DNS provider you use, find out the variable you need to provide to the LEGO, and provide them with secrets in both `signing_rsa_key` and `signing_ec_key` step in `.drone.yml`. \
   For example, LEGO wants `CLOUDNS_AUTH_ID` and `CLOUDNS_AUTH_PASSWORD` when using ClouDNS.
9. Append your own steps to push/publish/commit these resulting certificates, which named `rsa.cert.pem` and `ec.cert.pem` under the workspace. \
   **Attention! They will go to nowhere if you don't do anything.**