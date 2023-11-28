# Configuring Cloudflare SSL/TLS on Google App Engine 

Implementing end-to-end HTTPS encryption with CloudFlare for Google App Engine applications.

## Google App Engine - Custom Domains

### Add Domains

Register the root domain with Google Cloud Platform at the following:

```
https://console.cloud.google.com/appengine/settings/domains?project=<Project_Id>
```

## Cloudfare DNS

### Configure DNS Records for Google App Engine

Add a record for the root (`@`) or subdomain (`sub.domain.com`) pointing to Google Cloud Platform.

```
Type    Name    Target                  TTL     Proxy status
CNAME   sub     ghs.googlehosted.com    Auto    DNS-only
```

## Cloudfare SSL/TLS

### Encryption in __Full__ mode

Ensure your SSL/TLS encryption mode is set to _Full_ and not _Full (strict)_.

### Origin Certificates and Private Keys

Issue an _Origin Certificate_ for the root and wildcard (`*`) hostnames.

Navigate to __SSL/TLS__ -> Origin Server -> __Create Certificate__ and use the following configuration:

```
Private key type    Hostnames                  Certificate Validity
RSA                 domain.com,*.domain.com    15 years 
```

Using the `PEM (Default)` __Key format__; 
* Copy the _Origin Certificate_ into a `domain.com-YYYY-MM-dd.pem` file
* Copy the _Private key_ into a `domain.com-YYYY-MM-dd.key` file

Edit the _domain.com-YYYY-MM-dd.pem_ file and append the following [Cloudflare Origin CA root certificate](https://support.cloudflare.com/hc/en-us/articles/115000479507-What-are-the-root-certificate-authorities-CAs-used-with-CloudFlare-Origin-CA-#h_30cc332c-8f6e-42d8-9c59-6c1f06650639) after the newly created certificate:

 * [cloudflare_origin_ecc.pem](https://support.cloudflare.com/hc/article_attachments/360037898732/origin_ca_ecc_root.pem)
 
```
...
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
...
```

### Converting to RSA

Open a terminal with `OpenSSL` or install using the following (Mac OSX):

```sh
brew install openssl
```

Convert the private key to RSA with the following shell command:

```sh
openssl rsa -in domain.com-YYYY-MM-dd.key -out domain.com-RSA-YYYY-MM-dd.key
```

## Google App Engine - SSL Certificates

### Uploading the Certificate

Navigate to the following URL in Google Cloud Platform to __Upload a new certificate__:

```
https://console.cloud.google.com/appengine/settings/certificates?project=<Project_Id>
```

Provide a _Name_ for the certificate (e.g. `CF-YYYY-MM-DD`) and upload the certificate and key.
 * __PEM encoded X.509 public key certificate__: domain.com-YYYY-MM-dd.pem
 * __Unencrypted PEM encoded RSA private key__: domain.com-RSA-YYYY-MM-dd.key
 
### Assigning the Mapped Domains

After uploading, select the name of the newly added certificate (e.g. `CF-YYYY-MM-DD`)

Under __Enable SSL for the following custom domains__, select all domains that will use the corresponding certificate.

```
     Domain name
✓    *.domain.com
✓    sub.domain.com
```

## Cloudfare DNS - Enable Proxy

### Set Status to Proxied

Update the `CNAME` record to now be proxied through CloudFlare:

```
Type    Name    Target                  TTL     Proxy status
CNAME   sub     ghs.googlehosted.com    Auto    Proxied
```
