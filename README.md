
<img width="2664" height="1364" alt="Frame 240 (1)" src="https://github.com/user-attachments/assets/6a70ae47-0c5b-4f96-b23f-2623c2fe6ff7" />

# Spore Drive

**Spore Drive is a simple deployment tool for building a Taubyte cloud from your own servers.**

Turn a list of servers into a fully-configured Taubyte cloud with code. No manual setup. No control plane to manage.

## What it does

- Provisions multiple servers simultaneously
- Installs and configures Tau on each node
- Connects nodes into a meshed peer-to-peer network
- Agentless deployment using SSH

## Quick Start

**Install Spore Drive:**

```bash
npm install @taubyte/spore-drive typescript tsx --save-dev
```

**Basic Usage:**

```typescript
import { Config, Drive, TauLatest, CourseConfig } from "@taubyte/spore-drive";

const config = new Config("./config");
await config.init();

// Configure domains, SSH, hosts, and shapes
// ... (see documentation)

const drive = new Drive(config, TauLatest);
await drive.init();
const course = await drive.plot(new CourseConfig(["compute"]));
await course.displace();
```

**Configuration:**

1. Create a `hosts.csv` file with your servers:
   ```csv
   hostname,public_ip
   node1.example.com,203.0.113.1
   node2.example.com,203.0.113.2
   ```

2. Set up your `.env` file:
   ```bash
   SSH_KEY=ssh-key.pem
   SERVERS_CSV_PATH=hosts.csv
   SSH_USER=ssh-user
   ROOT_DOMAIN=example.com
   GENERATED_DOMAIN=g.example.com
   ```

## Documentation

For detailed installation instructions, configuration options, and deployment guides, see the [official Spore Drive documentation](https://tau.how/platform/spore-drive/).

## Server Requirements

- Linux servers with SSH access on port 22
- User account with sudo/root privileges
- SSH key authentication enabled

## Resources

- [Official Spore Drive Documentation](https://tau.how/platform/spore-drive/)
- [Boilerplate Example](https://github.com/taubyte/spore-drive-boilerplate)
- [Tau Documentation](https://tau.how)

## License

BSD-3-Clause
