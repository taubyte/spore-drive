
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

Node.js/TypeScript:
```bash
npm install @taubyte/spore-drive typescript tsx --save-dev
```

Python:
```bash
pip install spore-drive
```

## Documentation

For detailed installation instructions, configuration options, and deployment guides, see the [official Spore Drive documentation](https://tau.how/platform/spore-drive/).

## Server Requirements

- Linux servers with SSH access on port 22
- User account with sudo/root privileges
- SSH key authentication enabled

## How It Works

Spore Drive is an RPC service that runs on your local machine. It uses SSH to connect to your servers and deploy Tau without requiring agents on target machines. For implementation details, see the [Python client source code](https://github.com/taubyte/tau/tree/main/pkg/spore-drive/clients/py).

## Examples

- **TypeScript/Node.js**: [Spore Drive Boilerplate](https://github.com/taubyte/spore-drive-boilerplate)
- **Python**: [Python Client Example](https://github.com/taubyte/tau/tree/main/pkg/spore-drive/clients/py/example)

## Resources

- [Official Spore Drive Documentation](https://tau.how/platform/spore-drive/)
- [Boilerplate Example](https://github.com/taubyte/spore-drive-boilerplate)
- [Tau Documentation](https://tau.how)

## License

BSD-3-Clause
