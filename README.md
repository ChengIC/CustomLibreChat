# 1. Setup Environment and Configuration

## 1.1. Environment variables
```bash
# Environment variables
cp .env.example .env

# Docker compose override
cp docker-compose.override.yml.example docker-compose.override.yml

# librechat.yaml for custom interface, mcpServers, endpoints, etc.
cp librechat.example.yaml librechat.yaml
```

## 1.2. NPM
```bash
# Install dependencies
npm ci

# Local development with backend and frontend
npm run backend:dev

npm run frontend:dev

# Run backend and frontend
npm run backend

npm run frontend
```

## 1.3. Docker

### Docker for development
```bash
# Step 1: Development environment, build backend image
docker build -t librechat:dev .

# Step 2: Copy docker-compose.yml to docker-compose-dev.yml
# cp docker-compose.yml docker-compose-dev.yml

# Step 3: Go to docker-compose-dev.yml and change the "image: ghcr.io/danny-avila/librechat-dev-api:latest" to "image: librechat:dev" in line 12 (IMPORTANT!!!)

# Step 4: Run docker-compose-dev.yml
docker compose -f docker-compose-dev.yml up -d
```

#### Docker for production
```bash
# Step 1: Production environment, build backend and frontend images
docker build -f Dockerfile.multi -t librechat:latest .

# Step 2: Copy deploy-compose.yml to docker-compose-prod.yml
# cp deploy-compose.yml docker-compose-prod.yml

# Step 3: Go to docker-compose-prod.yml and change the "image: ghcr.io/danny-avila/librechat-dev-api:latest" to "image: librechat:latest" in line 7 (IMPORTANT!!!)

# Step 4: Run docker-compose-prod.yml
docker compose -f docker-compose-prod.yml up -d
```

# 2. Customize the frontend

## 2.1. Modify the app title, custom footer and FAQ URL
Modify the `.env` file.
```bash
APP_TITLE=<YOUR APP TITLE>
CUSTOM_FOOTER=<YOUR CUSTOM FOOTER>
HELP_AND_FAQ_URL=<YOUR HELP AND FAQ URL>
```

### 2.2. Modify the icon
In `client/src/index.html`, modify the title and icon in line 9-14.
```html
<meta name="description" content="LibreChat - An open source chat application with support for multiple AI models" />
<title>LibreChat</title>
<link rel="shortcut icon" href="#" />
<link rel="icon" type="image/png" sizes="32x32" href="/assets/favicon-32x32.png" />
<link rel="icon" type="image/png" sizes="16x16" href="/assets/favicon-16x16.png" />
<link rel="apple-touch-icon" href="/assets/apple-touch-icon-180x180.png" />
```

### 2.3. Modify the banner
In `client/src/components/Auth/AuthLayout.tsx`, modify the banner element as needed in line 65-85.
```tsx
{/* Original logo */}
<div className="mt-6 h-10 w-full bg-cover">
    <img
    src="/assets/logo.svg"
    className="h-full w-full object-contain"
    alt={localize('com_ui_logo', { 0: startupConfig?.appTitle ?? 'LibreChat' })}
    />
</div>

{/*
    Custom banner.
    Uncomment the below code and comment the above code to use.
    Change the src and adjust the height and width as needed.
*/}
{/* <div className="mt-6 h-10 w-full bg-cover">
    <img
    src="/assets/<YOUR BANNER>.png"
    style={{ height: '80px', width: '150%' }}
    className="object-contain"
    />
</div> */}
```

### 2.4. Modify the color theme
Unfortunatly, the color theme in this framework is not modularized. Basically, we need to find the html elements and modfiy them accordingly.

Most of the elements such as hover and button colors are defined in `client/src/style.css` and can be modified as needed as below.

For simplicity, I refer the original color variables and introduce a new color palette as blue-50 to blue-900 to replace the original gray-50 to gray-900 for the hover and button colors.

```css
--surface-hover: var(--blue-600);
--surface-hover-alt: var(--blue-600);
--surface-active: var(--blue-500);
--surface-active-alt: var(--blue-700);
```

### 2.5. Add extra menue
In `client/src/components/Nav`, I develped a `MenuSettings.tsx` component to add extra menue items and import this component in `Nav.tsx`.

### 2.6. Privacy Policy and Terms of Service
In `librechat.example.yaml`, the `privacyPolicy` and `termsOfService` settings can be defined as needed.

Make sure the `librechat.yaml` file is included in your docker compose file.


# 3. Deploy on Azure VM

## 3.1. Azure VM

- Create a resource group or use an existing one.

- Create a virtual machine and save the SSH key

- Connect to the virtual machine.
Use command `ssh -i <your-key-file-path> <username>@<public-ip-address>` to generate the SSH key. You can find your public IP address in the Azure portal, such as `13.80.10.10`.

- Add security settings to the virtual machine. (VERY IMPORTANT!!!). Please allow <b>port 80</b> and <b>port 443</b>, so you can set up the certificate and the web server.

## 3.2. Prepare the domain and set up the SSL. 

***Note: I am currently using the DNS server domain which can be configured on the Azure VM management portal.*** Please follow the official documentation to configure your own domain.

In the Azure VM, use the following command to install Certbot and set up the SSL.
```bash
# Install Certbot
sudo apt update
sudo apt install -y certbot python3-certbot-nginx

# start Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Obtain SSL certificate
sudo certbot --nginx -d yourdomain.com
```
You can also test the expiration date for this SSL via
```bash
sudo certbot renew --dry-run
```
After (a) and (b), you should be able to visit the original Nginx page via `https://yourdomain.com`. Then you can modify the Nginx configure file to redirect the traffic to the docker app service.

## 3.3. Run the app
- Install Docker. You should refer to official documentation to install Docker due to different versions of Ubuntu.

- Git clone this repository and checkout to the `custom` branch.
Follows the section 1.3 to build the docker image and use docker compose to run the container.

## 3.4. Set up the Nginx for the app
Access the file `/etc/nginx/sites-available/default` on Azure VM, you can use Vi editor or nano editor.

Replace the orginal with following contents. Please replace the `yourdomain.com` with your own domain.

```
server {
    if ($host = yourdomain.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


        listen 80 ;
        listen [::]:80 ;
    server_name yourdomain.com;
    return 404; # managed by Certbot


}

server {
    listen 443 ssl;
    server_name yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

}

server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$host$request_uri;
}
```

Then test and restart the Nginx
```bash
sudo nginx -t
sudo systemctl restart nginx
```