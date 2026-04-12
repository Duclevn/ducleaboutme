# Deploying `ducle.uk` to a VPS

This guide assumes:

- Your VPS runs Ubuntu.
- You will serve the site with Nginx.
- `audiobooks.ducle.uk` already has its own config and should stay untouched.
- As checked on April 12, 2026, `ducle.uk` and `audiobooks.ducle.uk` are both using Cloudflare DNS.
- The source code is hosted at: https://github.com/Duclevn/ducleaboutme.git

---

## First-time deployment

### 1. Update DNS for the main domain only

In Cloudflare DNS:

- Set the `A` record for `ducle.uk` to your VPS IPv4 address.
- If your VPS has IPv6, set the `AAAA` record for `ducle.uk` as well.
- Leave `audiobooks.ducle.uk` unchanged.

Recommended for the first deployment:

- Temporarily switch the `ducle.uk` DNS record to **DNS only** (not Proxied).
- After SSL is working, you can turn the Cloudflare proxy back on.

### 2. Install Nginx and Certbot on the VPS

```bash
sudo apt update
sudo apt install -y nginx certbot python3-certbot-nginx git
```

### 3. Clone the repo into the web root

```bash
sudo mkdir -p /var/www/ducle.uk
sudo chown -R $USER:$USER /var/www/ducle.uk
git clone https://github.com/Duclevn/ducleaboutme.git /var/www/ducle.uk/public
```

### 4. Add the Nginx site config

Copy the config from the repo:

```bash
sudo cp /var/www/ducle.uk/public/deploy/nginx/ducle.uk.conf /etc/nginx/sites-available/ducle.uk
sudo ln -s /etc/nginx/sites-available/ducle.uk /etc/nginx/sites-enabled/ducle.uk
```

If the default site is still enabled, remove it:

```bash
sudo rm -f /etc/nginx/sites-enabled/default
```

Test and reload Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### 5. Create the SSL certificate

```bash
sudo certbot --nginx -d ducle.uk
```

Choose the redirect option when Certbot asks whether HTTP should redirect to HTTPS.

### 6. Verify the deployment

Check these URLs in the browser:

- `http://ducle.uk` — should redirect to HTTPS
- `https://ducle.uk` — should show the personal website

If you turned Cloudflare proxy off earlier, switch it back on after HTTPS works correctly.
Use `Full (strict)` SSL mode in Cloudflare if the proxy is enabled.

---

## Deploying a new code commit (regular update)

After you push new changes to GitHub, SSH into the VPS and run:

```bash
cd /var/www/ducle.uk/public
git pull origin main
```

No build step is needed — this is a plain static site.
Nginx serves the files directly; there is no need to restart or reload Nginx.

### One-liner from your local machine (optional)

If you have SSH access set up, you can deploy in a single command from your Windows terminal:

```powershell
ssh user@YOUR_VPS_IP "cd /var/www/ducle.uk/public && git pull origin main"
```

Replace `user` and `YOUR_VPS_IP` with your actual SSH username and VPS IP address.

---

## Notes

- Do **not** merge the `ducle.uk` Nginx config into the `audiobooks.ducle.uk` server block. Keep them as separate files under `/etc/nginx/sites-available/`.
- If `ducle.uk` already points to the same VPS as `audiobooks.ducle.uk`, you only need to add the new site folder and Nginx config — the subdomain site is unaffected.
- The CV PDF (`20260402_SBA_Le_Minh_Duc_CV.docx.pdf`) is excluded from the repo via `.gitignore` and is not deployed.
