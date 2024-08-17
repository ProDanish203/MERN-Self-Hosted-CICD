# Self-Hosted CI/CD

- Configure Your VPS

## Create a new user

```
useradd -m -s /bin/bash username
groups username

# This will add the user to the sudoers group, giving them the ability to run commands with sudo privileges.
usermod -aG sudo username 

# Create password for the created user
sudo passwd username

# Switch to newly created user
su - username
```

## Create a new folder and cd into it
- Follow the steps that github action running provides with any user other than root

## Give access to user in that folder
```
sudo chmod -R 777 . 
```
- Follow the instructions and a runner will be added in your git repository with an offline status

### Activate the runner
```
sudo ./svc.sh install
sudo ./svc.sh start
```
- It will now show status of `Idle`
- Start the services with pm2 in the _work folder
- Remove password required for some scripts of the runner user
```
sudo visudo -f /etc/sudoers.d/username

your_runner_username ALL=(ALL) NOPASSWD: /usr/sbin/service nignx restart
```

## Create a workflow for CD
```
name: Continous Deployment

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  build-and-deploy:
    runs-on: self-hosted
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Setup Node.js 22.2.0
        uses: actions/setup-node@v2
        with:
          node-version: "22.2.0"

      - name: Install PM2 globally
        run: npm install -g pm2

      - name: Create server .env file
        run: |
          cd server
          echo "${{ secrets.SERVER_ENV }}" > .env

      - name: Create client .env file
        run: |
          cd client
          echo "${{ secrets.CLIENT_ENV }}" > .env

      - name: Install dependencies and build
        run: |
          cd server
          npm ci
          cd ../client
          npm ci
          npm run build
      - name: Start server and client
        run: |
          pm2 restart server
          pm2 restart client
      - name: Reload Nginx
        run: sudo service nginx restart
```