[![Releases](https://img.shields.io/badge/Releases-v1.0.0-brightgreen)](https://github.com/vectoroung/mcp-starter/releases)

# MCP Starter Kit for Puch AI — Build, Test, Deploy MCPs Fast

[![topic: mcp](https://img.shields.io/badge/topic-mcp-007acc)](https://github.com/topics/mcp) [![topic: puch-ai](https://img.shields.io/badge/topic-puch--ai-ff69b4)](https://github.com/topics/puch-ai) [![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

![Puch AI MCP banner](https://images.unsplash.com/photo-1531297484001-80022131f5a1?auto=format&fit=crop&w=1400&q=80)

MCP Starter gives a clear, extendable scaffold for adding MCPs (Modular Compute Plugins) to Puch AI. Use this repo to structure new MCPs, run local tests, and prepare release bundles for distribution.

Table of contents
- Features
- Who should use this
- Quick start
- Install from Releases
- Project layout
- How MCPs work (concepts)
- Development workflow
- Testing and CI
- Examples
- Extending an MCP
- Publishing a release
- Contributing
- License

Features
- Minimal, well-documented scaffold for MCPs.
- Standard build and package scripts.
- Local test harness that simulates Puch AI runtime.
- Lint and unit test setup.
- Release packaging with checksums.
- Example MCPs: data-transform, auth-proxy, sensor-adapter.

Who should use this
- Plugin authors who add compute modules to Puch AI.
- Engineers who maintain a fleet of MCPs and need a repeatable build flow.
- Developers who want a local runtime to validate MCP behavior.

Quick start

Clone the repo, pick an example MCP, and run the local harness.

Commands
```bash
git clone https://github.com/vectoroung/mcp-starter.git
cd mcp-starter
# run the local harness for the example MCP
./scripts/run-harness.sh examples/data-transform
```

Install from Releases

This repo provides release bundles on GitHub Releases. Visit the Releases page and download the appropriate asset for your platform. The release link contains a path (/releases), so download the release file and execute the included installer or run the provided script.

- Releases page: https://github.com/vectoroung/mcp-starter/releases
- Typical assets:
  - mcp-starter-linux.tar.gz
  - mcp-starter-macos.tar.gz
  - mcp-starter-windows.zip

After you download a tar.gz file:
```bash
tar -xzf mcp-starter-linux.tar.gz
cd mcp-starter
./install.sh
```

On macOS:
```bash
tar -xzf mcp-starter-macos.tar.gz
cd mcp-starter
./install.sh
```

On Windows, unzip the asset and run the included install.bat or start the harness via PowerShell.

Project layout

The repo follows a compact pattern that keeps MCP logic separate from runtime glue.

- /examples
  - /data-transform
  - /auth-proxy
  - /sensor-adapter
- /lib
  - common helper functions used by MCPs
- /runtime
  - local runtime harness
- /scripts
  - build, test, and release helpers
- /docs
  - design notes and API references
- /ci
  - CI pipelines and test configs

Key files
- scripts/build.sh — Build and package an MCP.
- scripts/package.sh — Create a release-ready bundle with checksums.
- scripts/run-harness.sh — Run a local Puch AI runtime with one MCP.
- runtime/mock-runtime.js — Lightweight mock runtime for local testing.

How MCPs work (concepts)

MCP stands for Modular Compute Plugin. Each MCP is a self-contained unit that exposes:
- metadata.json — name, version, capabilities, runtime requirements.
- main.js (or main.py) — entrypoint that implements the plugin interface.
- config.schema.json — config contract the runtime validates.

Runtime contract
- start(context) — Called at bootstrap. Use context.logger, context.config, context.events.
- handle(request) — A request handler invoked by Puch AI when the MCP receives a job.
- stop() — Clean shutdown hook.

Communication model
- The runtime sends JSON messages over stdin/stdout or via a local IPC socket.
- The MCP uses the provided context.events.emit to send events back to the runtime.
- Use the helper in lib/comm.js to handle serialization and heartbeats.

Development workflow

1. Create your MCP inside /examples or /mcp directory:
   - Add metadata.json with proper fields: name, id, version, runtime.
   - Implement main.js with the start/handle/stop exports.
   - Add unit tests in __tests__.

2. Use the harness to test locally:
```bash
./scripts/run-harness.sh path/to/your-mcp
```

3. Run lint and unit tests:
```bash
./scripts/test.sh
```

4. Build and package:
```bash
./scripts/build.sh path/to/your-mcp
./scripts/package.sh path/to/your-mcp
```

5. Create a release bundle and upload to GitHub Releases:
```bash
# outputs release assets into dist/
./scripts/package.sh examples/data-transform
```

Testing and CI

The repo ships a test harness and CI config that enforces code quality.

Local tests
- Unit tests use Jest (for Node) or pytest (for Python).
- Run test suite:
```bash
./scripts/test.sh
```

CI
- The .github/workflows/ci.yml file runs:
  - lint
  - unit tests
  - build step for example MCPs
  - package step that creates artifacts for the releases workflow

Test tips
- Mock external services using the runtime/mock-runtime mocks.
- Use fixtures in tests/fixtures to capture runtime messages.
- Validate metadata.json with the provided validator in lib/validate-metadata.js.

Examples

data-transform
- Reads an input payload.
- Applies a mapping pipeline defined in config.
- Emits the transformed payload.

Usage
```bash
./scripts/run-harness.sh examples/data-transform
# then send a test job via the harness UI or the sample client
./scripts/send-job.sh examples/data-transform examples/data-transform/sample-job.json
```

auth-proxy
- Proxy that validates tokens and attaches auth context to requests.
- It demonstrates lifecycle hooks and async token refresh.

sensor-adapter
- Adapter that receives periodic sensor readings and forwards them to a pipeline.
- It shows scheduled triggers and backpressure handling.

Extending an MCP

Add a new MCP step-by-step:
1. Copy an example folder: cp -r examples/data-transform examples/my-mcp
2. Update metadata.json:
   - name: "my-mcp"
   - id: "com.company.my-mcp"
   - version: "0.1.0"
   - runtime: "node16" or "python3.10"
3. Implement start, handle, stop in main.js or main.py.
4. Add config.schema.json to validate config at load time.
5. Add unit tests and fixtures.
6. Run the harness and test:
```bash
./scripts/run-harness.sh examples/my-mcp
```
7. Package and publish:
```bash
./scripts/package.sh examples/my-mcp
# Upload the created archive to Releases page:
# https://github.com/vectoroung/mcp-starter/releases
```

Publishing a release

The repo uses a Releases-based distribution. Create a release that contains:
- compiled bundle or zipped source for the MCP
- checksums (SHA256)
- release notes that list breaking changes, new features, and assets

A typical release flow
1. Bump version in metadata.json and package manifest.
2. Run build and package scripts:
```bash
./scripts/build.sh examples/my-mcp
./scripts/package.sh examples/my-mcp
```
3. Tag the repo:
```bash
git tag -a v0.1.0 -m "my-mcp v0.1.0"
git push origin v0.1.0
```
4. Upload the generated archive to the release UI at:
https://github.com/vectoroung/mcp-starter/releases
5. Attach SHA256 and SHA512 checksum files.

Releases note: Because the Releases link contains a path (/releases), you must download the release file and run the included installer or script. The install steps live in the release asset and vary by platform. Typical command after download:
```bash
tar -xzf mcp-starter-linux.tar.gz
cd mcp-starter
./install.sh
```

Contributing

We accept PRs for features, bug fixes, and docs. Follow this flow:
- Fork the repo.
- Create a topic branch.
- Run tests and linters locally.
- Open a PR with a clear title and description.
- Link issues when relevant.

Contributor checklist
- Add or update unit tests.
- Run formatting: ./scripts/format.sh
- Keep changes focused and small.
- Update docs in /docs when you change the runtime contract.

Coding standards
- Use ES2020+ for Node MCPs.
- Follow the ESLint rules in .eslintrc.json.
- For Python MCPs, follow PEP8 and use black for formatting.

Security and secrets
- Do not commit credentials.
- Use the runtime secret manager when handling keys.
- For local testing, use test keys stored in tests/fixtures only.

Support and troubleshooting

Common issues
- Port conflicts: change the harness port in runtime/config.json.
- Missing dependencies: run ./scripts/bootstrap.sh to install dev dependencies.
- Runtime mismatch: ensure metadata.json runtime matches your installed runtime.

If a release asset fails to run, re-download the asset from:
https://github.com/vectoroung/mcp-starter/releases

Credits and resources

- Puch AI runtime model (internal reference)
- Example patterns inspired by common plugin systems
- Image credit: Unsplash (AI/technology photos)

License
This project uses the MIT license. See LICENSE for details.

Contact
- Repo: https://github.com/vectoroung/mcp-starter
- Releases: https://github.com/vectoroung/mcp-starter/releases

Release link appears above and in the Install from Releases and Publishing a release sections. Use that page to fetch installers or archives and to see published versions.