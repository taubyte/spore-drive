
<img width="2664" height="1364" alt="Frame 240 (1)" src="https://github.com/user-attachments/assets/6a70ae47-0c5b-4f96-b23f-2623c2fe6ff7" />

# Spore Drive

**Spore Drive is a simple deployment tool for building a Taubyte cloud from your own servers.**

Turn a list of servers into a fully-configured Taubyte cloud with code. No manual setup. No control plane to manage.



## What it does

- Provisions multiple servers simultaneously
- Installs and configures Tau on each node
- Connects nodes into a meshed peer-to-peer network
- Agentless deployment using SSH



## Create Your Own Implementation

This tutorial walks you through building your own Spore Drive deployment script from scratch, following the boilerplate approach.

### Prerequisites

- Node.js 18+ and npm
- TypeScript
- SSH access to your target servers
- A root domain for your cloud platform

### Step 1: Project Setup

Initialize a new Node.js project:

```bash
mkdir my-cloud-deployment
cd my-cloud-deployment
npm init -y
```

Install TypeScript and the Spore Drive SDK:

```bash
npm install typescript tsx @taubyte/spore-drive --save-dev
npx tsc --init
```

### Step 2: Configure TypeScript

Update `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node",
    "declaration": true,
    "outDir": "dist",
    "rootDir": ".",
    "strict": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "skipLibCheck": true,
    "lib": ["ES2020"]
  },
  "include": ["src/**/*.ts"],
  "exclude": ["node_modules"]
}
```

Add a deploy script to `package.json`:

```json
{
  "scripts": {
    "displace": "tsx src/index.ts"
  }
}
```

### Step 3: Prepare Server List

Create a CSV file (default: `hosts.csv`) with your server information:

```csv
hostname,public_ip
node1.example.com,203.0.113.1
node2.example.com,203.0.113.2
```

The CSV file should contain:
- **hostname**: The fully qualified domain name of your server
- **public_ip**: The public IP address of your server

### Step 4: Environment Configuration

Create a `.env` file in your project root and configure it step by step:

**1. Server Configuration**

Configure SSH access to your servers:

```bash
SSH_KEY=ssh-key.pem
SERVERS_CSV_PATH=hosts.csv
SSH_USER=ssh-user
```

- **SSH_KEY**: Path to your SSH private key file (e.g., `ssh-key.pem`, `id_rsa`). This is the private key that corresponds to the public key installed on your servers.
- **SERVERS_CSV_PATH**: Path to your CSV file containing server information (default: `hosts.csv`).
- **SSH_USER**: The username for SSH connections (e.g., `ubuntu`, `root`, `ssh-user`). Must match the user that has the public key installed.

**About SSH Keys:**
- Spore Drive uses SSH to connect to your servers without installing agents
- Common key formats: `.pem` (common with AWS EC2), `id_rsa`, `id_ed25519`
- Ensure your private key file has correct permissions: `chmod 400 ssh-key.pem`
- The corresponding public key must be in `~/.ssh/authorized_keys` on each target server
- Your SSH user must have sudo/root privileges for installation

**2. Domain Configuration**

Set your domain names for the cloud platform:

```bash
ROOT_DOMAIN=example.com
GENERATED_DOMAIN=g.example.com
```

- **ROOT_DOMAIN**: Your primary domain name for the cloud platform.
- **GENERATED_DOMAIN**: Subdomain where auto-generated services will be accessible.

**3. Namecheap DNS Configuration (Optional)**

If you're using Namecheap for DNS management, add these optional variables:

```bash
NAMECHEAP_API_KEY=your_api_key
NAMECHEAP_IP=your_ip
NAMECHEAP_USERNAME=your_username
```

- **NAMECHEAP_API_KEY**: Your Namecheap API key for DNS management.
- **NAMECHEAP_IP**: Your IP address registered with Namecheap.
- **NAMECHEAP_USERNAME**: Your Namecheap account username.

Leave these empty if you're not using Namecheap DNS.

**4. Secure Your Configuration**

Add `.env` and sensitive files to your `.gitignore` to prevent committing secrets:

```bash
echo ".env" >> .gitignore
echo "*.pem" >> .gitignore
echo "id_rsa" >> .gitignore
echo "id_ed25519" >> .gitignore
echo "hosts.csv" >> .gitignore
```

### Step 5: Write Your Deployment Code

Create `src/index.ts` and build it step by step:

**1. Imports and Setup**

First, import the necessary modules and set up the main function:

```typescript
import {
  Config,
  CourseConfig,
  Drive,
  TauLatest,
} from "@taubyte/spore-drive";
import * as fs from "fs/promises";
import * as path from "path";

async function main() {
```

**2. Initialize Configuration**

Create a config object that stores settings in a local directory:

```typescript
  const config = new Config(`${__dirname}/../config`);
  await config.init();
```

**3. Configure Domains**

Set your root domain and generated subdomain, then generate cryptographic keys:

```typescript
  const domain = config.Cloud().Domain();
  await domain.Root().Set(process.env.ROOT_DOMAIN || "pom.ac");
  await domain.Generated().Set(process.env.GENERATED_DOMAIN || "g.pom.ac");
  await domain.Validation().Generate();
  await config.Cloud().P2P().Swarm().Generate();
```

**4. Configure SSH**

Set up SSH access using your username and private key path from the `.env` file:

```typescript
  const ssh = config.Cloud().SSH();
  await ssh.User().Set(process.env.SSH_USER || "ssh-user");
  await ssh.Key().Set(process.env.SSH_KEY || "ssh-key.pem");
```

**5. Read Servers from CSV File**

Parse the `hosts.csv` file to get the list of servers with hostnames and IP addresses:

```typescript
  const csvPath = process.env.SERVERS_CSV_PATH || "hosts.csv";
  const csvContent = await fs.readFile(csvPath, "utf-8");
  const lines = csvContent.trim().split("\n");
  const servers = lines.slice(1).map((line) => {
    const [hostname, public_ip] = line.split(",");
    return { hostname: hostname.trim(), public_ip: public_ip.trim() };
  });
```

**6. Add Hosts to Configuration**

Register each server from the CSV file into the Spore Drive configuration:

```typescript
  const hosts = config.Cloud().Hosts();
  for (const server of servers) {
    await hosts.Add(server.hostname, server.public_ip);
  }
```

**7. Save Configuration**

Commit the configuration to disk before proceeding with deployment:

```typescript
  await config.Save();
```

**8. Initialize Drive**

Create a drive instance with the latest Tau binary source:

```typescript
  const drive = new Drive(config, new TauLatest());
  await drive.init();
```

**9. Plot Course**

Plan the deployment steps with concurrency and timeout settings:

```typescript
  const courseConfig: CourseConfig = {
    concurrency: 2,
    timeout: 600,
  };
  const course = await drive.plot(courseConfig);
```

**10. Deploy**

Execute the deployment to all servers via SSH:

```typescript
  await course.displace();
  console.log("Deployment complete.");
}

main().catch(console.error);
```

### Step 6: Deploy

Make sure your `.env` file is configured and your SSH key file is in place:

```bash
# Verify your .env file
cat .env

# Ensure SSH key has correct permissions
chmod 400 ssh-key.pem

# Verify hosts.csv exists and is formatted correctly
cat hosts.csv
```

Run your deployment:

```bash
npm run displace
```

Spore Drive will:
1. Connect to each server via SSH using your `.pem` key
2. Install Tau binary on each node
3. Configure services and networking
4. Join nodes into a P2P mesh
5. Complete the cloud setup

## Server Requirements

- Linux servers with SSH access on port 22
- User account (specified in `SSH_USER`) with:
  - sudo/root privileges
  - SSH key authentication enabled



## How It Works

1. **Config**: Defines your cloud structure (domains, hosts, services)
2. **Drive**: Orchestrates the deployment process
3. **Course**: Plans the deployment steps
4. **Displace**: Executes deployment via SSH to all hosts

The Spore Drive service runs locally and uses SSH to deploy Tau across your server fleet without requiring agents on target machines.



## Resources

- [Official Spore Drive Documentation](https://tau.how/platform/spore-drive)
- [Boilerplate Example](https://github.com/taubyte/spore-drive-boilerplate)
- [Tau Documentation](https://tau.how)



## License

BSD-3-Clause
