# Rails + PostgreSQL CodeSpace Setup Guide

This guide will help you set up a Rails API with PostgreSQL using GitHub CodeSpaces to avoid local environment issues.

## Prerequisites

- A GitHub account
- Basic knowledge of Rails and Git

## Step 1: Create Project Directory

### 1.1 Create a new project folder
```bash
mkdir my-rails-codespace-project
cd my-rails-codespace-project
```

### 1.2 Initialize Git repository
```bash
git init
git branch -M main
```

## Step 2: Create DevContainer Configuration

### 2.1 Create the `.devcontainer` folder
```bash
mkdir .devcontainer
```

### 2.2 Create `devcontainer.json`
Create a file named `devcontainer.json` inside the `.devcontainer` folder with the following content:

```json
{
  "name": "Rails + Postgres",
  "dockerComposeFile": ["../docker-compose.yml"],
  "service": "app",
  "runServices": ["db"],
  "forwardPorts": [3000, 5432],
  "portsAttributes": { "3000": { "label": "Rails" } },
  "postCreateCommand": "gem install bundler -N || true"
}
```

### 2.3 Create `Dockerfile`
Create another file inside the `.devcontainer` folder named `Dockerfile` with the following content:

```dockerfile
FROM mcr.microsoft.com/devcontainers/ruby:1-3.3-bullseye

RUN apt-get update && \
    apt-get install -y build-essential libpq-dev curl git && \
    curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && \
    apt-get install -y nodejs && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /workspaces
```

## Step 3: Create Docker Compose Configuration

Create a file called `docker-compose.yml` in the **root directory** (outside of the `.devcontainer` folder):

```yaml
version: "3.8"
services:
  app:
    build:
      context: .
      dockerfile: .devcontainer/Dockerfile
    command: sleep infinity
    environment:
      POSTGRES_HOST: db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: app_development
    depends_on:
      - db
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
```

## Step 4: Push to GitHub and Create CodeSpace

1. **Create a new repository on GitHub:**
   - Go to [GitHub.com](https://github.com) and click "New repository"
   - Name your repository (e.g., "my-rails-codespace-project")
   - Make it public or private (your choice)
   - **Do NOT** initialize with README, .gitignore, or license (since we already have files)
   - Click "Create repository"

2. **Connect your local repository to GitHub:**
   ```bash
   git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPOSITORY_NAME.git
   ```

3. **Commit and push your code to GitHub:**
   ```bash
   git add .
   git commit -m "Add DevContainer configuration"
   git push -u origin main
   ```

4. **Open your repository on GitHub**

5. **Create a CodeSpace:**
   - Click the green **"Code"** button
   - Click **"CodeSpaces"** tab
   - Click **"Create CodeSpace"**
   - Wait for the environment to build (this may take a few minutes)

## Step 5: Set Up Rails Application

Once your CodeSpace is ready:

### 5.1 Install Rails
```bash
gem install rails
```

### 5.2 Create Rails API
```bash
rails new <your_app_name> --api --database=postgresql
cd <your_app_name>
```

### 5.3 Configure Database
Find the `config/database.yml` file and replace its content with:

```yaml
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS", 5) %>
  host: <%= ENV.fetch("POSTGRES_HOST", "db") %>
  port: <%= ENV.fetch("POSTGRES_PORT", 5432) %>
  username: <%= ENV.fetch("POSTGRES_USER", "postgres") %>
  password: <%= ENV.fetch("POSTGRES_PASSWORD", "postgres") %>

development:
  <<: *default
  database: <%= ENV.fetch("POSTGRES_DB", "app_development") %>

test:
  <<: *default
  database: <%= ENV.fetch("POSTGRES_DB_TEST", "app_test") %>

production:
  <<: *default
  database: <%= ENV.fetch("POSTGRES_DB", "app_production") %>
```

### 5.4 Configure Development Environment
Find `config/environments/development.rb` and add this line:

```ruby
config.hosts.clear
```

## Step 6: Standard Rails Development

From here, you can use Rails as normal:

```bash
# Create and migrate database
rails db:create
rails db:migrate

# Generate models
rails generate model YourModel

# Generate controllers
rails generate controller YourController

# Start the server
rails server
```

## Step 7: Accessing Your Application

**Important:** Your Rails server won't be accessible at `localhost:3000` in CodeSpaces.

To access your application:
1. Click on the **"PORTS"** tab in your terminal
2. Find the port labeled **"Rails (3000)"**
3. Hover over it and click the **"Open in Browser"** button (globe icon)

## Step 8: Testing Your API with HTTPie

Since you're used to using the HTTPie GUI application locally, you'll need to use the HTTPie CLI inside the CodeSpace instead.

### 8.1 Install HTTPie CLI
```bash
sudo apt-get update && sudo apt-get install -y httpie
```

### 8.2 Using HTTPie CLI
Instead of sending requests from your local HTTPie (which would need the forwarded URL), use HTTPie directly inside the container:

```bash
# CREATE
http POST :3000/products chef="Ron" instruction="Some instructions" ingredients="Some ingredients" cook_time:=10

# READ (index)
http GET :3000/products

# READ (show)
http GET :3000/products/1

# UPDATE
http PATCH :3000/products/1 chef="Updated Ron"

# DELETE
http DELETE :3000/products/1
```

**Note:** The colon shorthand `:3000` talks directly to the Rails server running inside the CodeSpace container, without needing to go through the forwarded URL. This is faster and more reliable than using the external URL.

## Troubleshooting

- If you encounter database connection issues, ensure the PostgreSQL service is running
- Make sure all environment variables are properly set in the `docker-compose.yml`
- If the CodeSpace fails to build, check the DevContainer configuration files for syntax errors

## File Structure

Your project should look like this:
```
your-project/
├── .devcontainer/
│   ├── devcontainer.json
│   └── Dockerfile
├── docker-compose.yml
└── README.md
```

After creating the Rails app:
```
your-project/
├── .devcontainer/
├── your_app_name/
│   ├── config/
│   │   ├── database.yml
│   │   └── environments/development.rb
│   └── ... (other Rails files)
├── docker-compose.yml
└── README.md
```

## What Are These Technologies?

### GitHub CodeSpaces
CodeSpaces is GitHub's cloud-based development environment. Instead of setting up Rails, PostgreSQL, and all dependencies on your local computer, CodeSpaces creates a virtual machine in the cloud with everything pre-configured. You access it through your web browser, and it works just like VS Code.

### Docker
Docker is a tool that packages applications and their dependencies into "containers" - think of them as lightweight virtual machines. This ensures your Rails app runs the same way everywhere, regardless of the host operating system.

### DevContainer
A DevContainer is a Docker container specifically designed for development. It includes all the tools, languages, and dependencies needed for your project. VS Code and CodeSpaces can automatically set up and connect to these containers.

## What Do Our Files Do?

### `devcontainer.json`
This file tells CodeSpaces how to set up your development environment:
- **name**: What to call your container
- **dockerComposeFile**: Points to our Docker Compose configuration
- **service**: Which service from docker-compose to use as the main development container
- **forwardPorts**: Makes ports 3000 (Rails) and 5432 (PostgreSQL) accessible
- **postCreateCommand**: Runs commands after the container is created

### `Dockerfile`
This file defines what goes into our development container:
- **FROM**: Starts with a pre-built Ruby container from Microsoft
- **RUN**: Installs additional packages we need (PostgreSQL client, Node.js, etc.)
- **WORKDIR**: Sets the working directory inside the container

### `docker-compose.yml`
This file defines multiple services that work together:
- **app**: Our main Rails development container (built from the Dockerfile)
- **db**: A PostgreSQL database container
- **environment**: Sets up database connection variables
- **volumes**: Ensures database data persists between container restarts

Together, these files create a complete, isolated development environment that works the same for everyone, eliminating "it works on my machine" problems!