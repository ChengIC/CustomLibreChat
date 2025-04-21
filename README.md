## Environment and configuration
```bash
# Environment variables
cp .env.example .env

# Docker compose override
cp docker-compose.override.yml.example docker-compose.override.yml

# librechat.yaml for custom interface, mcpServers, endpoints, etc.
cp librechat.example.yaml librechat.yaml
```

## NPM
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

## Docker

### Docker for development
```bash
# Step 1: Development environment, build backend image
docker build -t librechat:dev .

# Step 2: Copy docker-compose.yml to docker-compose-dev.yml
cp docker-compose.yml docker-compose-dev.yml

# Step 3: Go to docker-compose-dev.yml and change the "image: ghcr.io/danny-avila/librechat-dev-api:latest" to "image: librechat:dev" in line 12 (IMPORTANT!!!)

# Step 4: Run docker-compose-dev.yml
docker compose -f docker-compose-dev.yml up -d
```

### Docker for production
```bash
# Step 1: Production environment, build backend and frontend images
docker build -f Dockerfile.multi -t librechat:latest .

# Step 2: Copy deploy-compose.yml to docker-compose-prod.yml
cp deploy-compose.yml docker-compose-prod.yml

# Step 3: Go to docker-compose-prod.yml and change the "image: ghcr.io/danny-avila/librechat-dev-api:latest" to "image: librechat:latest" in line 7 (IMPORTANT!!!)

# Step 4: Run docker-compose-prod.yml
docker compose -f docker-compose-prod.yml up -d
```

## Custom elements
### Modify the app title, custom footer and FAQ URL
Modify the `.env` file.
```bash
APP_TITLE=<YOUR APP TITLE>
CUSTOM_FOOTER=<YOUR CUSTOM FOOTER>
HELP_AND_FAQ_URL=<YOUR HELP AND FAQ URL>
```

### Modify the icon
In `client/src/index.html`, modify the title and icon in line 9-14.
```html
<meta name="description" content="LibreChat - An open source chat application with support for multiple AI models" />
<title>LibreChat</title>
<link rel="shortcut icon" href="#" />
<link rel="icon" type="image/png" sizes="32x32" href="/assets/favicon-32x32.png" />
<link rel="icon" type="image/png" sizes="16x16" href="/assets/favicon-16x16.png" />
<link rel="apple-touch-icon" href="/assets/apple-touch-icon-180x180.png" />
```

### Modify the banner
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

### Modify the color theme
Unfortunatly, the color theme in this framework is not modularized. Basically, we need to find the html elements and modfiy them accordingly.

Most of the elements such as hover and button colors are defined in `client/src/style.css` and can be modified as needed as below.

For simplicity, I refer the original color variables and introduce a new color palette as blue-50 to blue-900 to replace the original gray-50 to gray-900 for the hover and button colors.

```css
--surface-hover: var(--blue-600);
--surface-hover-alt: var(--blue-600);
--surface-active: var(--blue-500);
--surface-active-alt: var(--blue-700);
```

### Add extra menue
In `client/src/components/Nav`, I develped a `MenuSettings.tsx` component to add extra menue items and import this component in `Nav.tsx`.

### Privacy Policy and Terms of Service
In `librechat.example.yaml`, the `privacyPolicy` and `termsOfService` settings can be defined as needed.

Make sure the `librechat.yaml` file is included in your docker compose file.