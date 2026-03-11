# cc-edge-claude-code-otel

Cribl Edge pack for Claude Code telemetry collection (session logs, OTLP).

## Version Policy

This pack uses **semantic versioning** anchored to the VisiCore upstream.
The JacobPEvans fork tracks the upstream `VisiCore/cc-edge-claude-code-otel` and publishes independent releases.

### Rules for AI agents

- **Patch bumps** (`X.Y.Z → X.Y.Z+1`): AI may bump for bug fixes, performance improvements, and minor corrections.
- **Minor bumps** (`X.Y.z → X.Y+1.0`): AI may bump for new features, new inputs, or non-breaking changes.
- **Major bumps** (`X.y.z → X+1.0.0`): **Human only.** Never bump the major version without explicit human approval.

### Release workflow

1. Make changes on a feature branch (pushing to the `fork` remote: `JacobPEvans/cc-edge-claude-code-otel`)
2. Update `package.json` version (minor or patch only)
3. Update the `## Release Notes` section in `README.md` with the new version entry
4. Merge PR to main on the fork
5. Create GitHub release with the `.crbl` artifact built locally:
   ```sh
   tar -czf cc-edge-claude-code-otel-vX.Y.Z.crbl data default package.json README.md
   cp cc-edge-claude-code-otel-vX.Y.Z.crbl cc-edge-claude-code-otel.crbl
   gh release create vX.Y.Z --draft \
     --repo JacobPEvans/cc-edge-claude-code-otel \
     cc-edge-claude-code-otel-vX.Y.Z.crbl cc-edge-claude-code-otel.crbl
   gh release edit vX.Y.Z --repo JacobPEvans/cc-edge-claude-code-otel --draft=false
   ```
   Note: Create as draft with assets first, then publish. Publishing before uploading assets makes the release immutable.
6. Update `install-packs.sh` in `kubernetes-monitoring` to reference the new version URL

## File Operations

- Read files with the Read tool (NEVER cat, head, tail)
- Edit existing files with the Edit tool (NEVER sed, awk)
- Create new files with the Write tool (NEVER echo >, heredocs)
- Search file contents with the Grep tool
- Find files with the Glob tool
