sudo apt update && sudo apt install certbot python3-certbot-dns-cloudflare

# Using acme.sh to Generate Let's Encrypt Certificates for Unifi UDM (with Cloudflare DNS)

## 1. Install acme.sh

```
curl https://get.acme.sh | sh
```

After installation, reload your shell or run:

```
source ~/.bashrc
```

## 2. Set up Cloudflare API credentials


You can use your existing Cloudflare API token file (e.g., `~/.secrets/cloudflare.ini`).

Example `~/.secrets/cloudflare.ini`:
```
# Cloudflare API token used by Certbot or acme.sh
dns_cloudflare_api_token = <your_token_here>
```

To use this file with acme.sh, set the environment variable before running acme.sh commands:

```
export CF_Token=$(grep dns_cloudflare_api_token ~/.secrets/cloudflare.ini | cut -d'=' -f2 | xargs)
```

Or, you can manually copy the token value and run:
```
export CF_Token="<your_token_here>"
```

acme.sh will use this environment variable for DNS validation.

## 3. Issue a certificate for your domain


### a. Register your account with Let's Encrypt

You only need to do this once:

```
acme.sh --register-account -m <replace_with_your_email> --server letsencrypt
```

### b. Issue a certificate for `ui.mockus.me`

```
acme.sh --issue --dns dns_cf -d ui.mockus.me --server letsencrypt
```

This will:
- Use your exported Cloudflare API token for DNS validation
- Request a certificate for `ui.mockus.me` from Let's Encrypt
- Save the certificate and key in `~/.acme.sh/ui.mockus.me/`

If successful, you will see output indicating the certificate and key file locations.

## 4. Upload certificate to Unifi UDM

1. Open your browser and go to:
	```
	https://<unifi_udm_ip>/network/default/settings/control-plane/console
	```
2. Go to **Certificates > Add New**.
3. Upload the following files:
	- **Certificate**: `fullchain.cer` (from `~/.acme.sh/ui.mockus.me_ecc/fullchain.cer`)
	- **Private Key**: `ui.mockus.me.key` (from `~/.acme.sh/ui.mockus.me_ecc/ui.mockus.me.key`)
4. Save and activate the certificate.
5. Your Unifi UDM should now use the new certificate for your custom domain.


---

## TODO: Automate Certificate Upload (Python)

Next time you need to update the certificate, automate the upload to Unifi UDM using Python and the Unifi API.

- Research or implement a Python script to:
	1. Authenticate to the UDM API
	2. Upload the new certificate and private key (likely as PEM or PFX)
	3. Activate the new certificate
- Reference community scripts or API docs for details
- Integrate this step into the renewal workflow for full automation

This will save time and enable unattended certificate renewals in the future.

