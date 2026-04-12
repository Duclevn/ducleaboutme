# Deploying `ducle.uk` to a VPS

This guide assumes:

- Your VPS runs Ubuntu or Debian.
- You will serve the site with Nginx.
- `audiobooks.ducle.uk` already has its own config and should stay untouched.
- As checked on April 12, 2026, `ducle.uk` and `audiobooks.ducle.uk` are both using Cloudflare DNS.

## 1. Update DNS for the main domain only

In Cloudflare DNS:

- Set the `A` record for `ducle.uk` to your VPS IPv4 address.
- If your VPS has IPv6, set the `AAAA` record for `ducle.uk` as well.
- Leave `audiobooks.ducle.uk` unchanged.

Recommended for the first deployment:

- Temporarily switch the `ducle.uk` DNS record to `DNS only` instead of `Proxied`.
- After SSL is working, you can turn the Cloudflare proxy back on if you want.

## 2. Install Nginx and Certbot on the VPS

```bash
sudo apt update
sudo apt install -y nginx certbot python3-certbot-nginx
```

## 3. Create a folder for the site

```bash
sudo mkdir -p /var/www/ducle.uk/public
sudo chown -R $USER:$USER /var/www/ducle.uk
```

## 4. Upload the static files from your Windows machine

Run these commands from PowerShell on your local computer:

```powershell
scp "H:\My Drive\AI\personalWebsite\index.html" user@YOUR_VPS_IP:/var/www/ducle.uk/public/
scp "H:\My Drive\AI\personalWebsite\styles.css" user@YOUR_VPS_IP:/var/www/ducle.uk/public/
```

Replace:

- `user` with your SSH username
- `YOUR_VPS_IP` with your VPS public IP

## 5. Add the Nginx site config

Copy the sample config in [deploy/nginx/ducle.uk.conf](/h:/My%20Drive/AI/personalWebsite/deploy/nginx/ducle.uk.conf) to the VPS:

```bash
sudo cp /path/to/ducle.uk.conf /etc/nginx/sites-available/ducle.uk
sudo ln -s /etc/nginx/sites-available/ducle.uk /etc/nginx/sites-enabled/ducle.uk
```

If your server still has the default site enabled, remove it:

```bash
sudo rm -f /etc/nginx/sites-enabled/default
```

Test and reload Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

## 6. Create the SSL certificate

```bash
sudo certbot --nginx -d ducle.uk
```

Choose the redirect option when Certbot asks whether HTTP should redirect to HTTPS.

## 7. Verify the deployment

Check these URLs in the browser:

- `http://ducle.uk`
- `https://ducle.uk`

If you turned Cloudflare proxy off earlier, you can switch it back on after the HTTPS site works correctly.

## 8. Updating the site later

Whenever you change the page, upload the files again:

```powershell
scp "H:\My Drive\AI\personalWebsite\index.html" user@YOUR_VPS_IP:/var/www/ducle.uk/public/
scp "H:\My Drive\AI\personalWebsite\styles.css" user@YOUR_VPS_IP:/var/www/ducle.uk/public/
```

No build step is needed because this is a plain static site.

## Notes

- Do not merge this config into the `audiobooks.ducle.uk` server block. Keep main domain and subdomain as separate Nginx site files.
- If `ducle.uk` already points to the same VPS as `audiobooks.ducle.uk`, you may only need to add the new site folder and Nginx config.
- If Cloudflare is enabled and you prefer Cloudflare SSL modes, use `Full (strict)` after Certbot is installed on the VPS.
