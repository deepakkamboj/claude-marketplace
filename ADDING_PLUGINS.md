# Adding New Plugins to DevBuild Studio Marketplace

This repository is structured as a **multi-plugin marketplace**. Follow this guide when adding new plugins.

## Current Plugin Structure

```
e:\git\claude-accessiblity\
├── .claude-plugin/
│   └── marketplace.json          # Marketplace definition (all plugins registered here)
├── plugins/
│   └── accessibility/            # Plugin #1
│       ├── agents/               # Autonomous agents
│       ├── skills/               # User-invoked skills
│       ├── mcp-server/          # MCP server (optional)
│       ├── docs/                # Plugin-specific docs
│       └── .mcp.json            # MCP configuration (optional)
├── ACCESSIBILITY_STANDARDS.md   # Shared resource (can be used by multiple plugins)
├── README.md                     # Marketplace README
└── USAGE.md                      # Usage guide
```

## Adding a New Plugin

### Step 1: Create Plugin Directory

Create the plugin directory structure under `plugins/`:

```bash
# Example: Adding a "performance" plugin
mkdir -p plugins/performance/agents
mkdir -p plugins/performance/skills
mkdir -p plugins/performance/docs
```

### Step 2: Add Plugin Files

Create the necessary files for your plugin:

```
plugins/performance/
├── agents/
│   └── optimizer.md              # Agent definition
├── skills/
│   └── bundle-analyzer/
│       └── SKILL.md             # Skill definition
├── docs/
│   └── PERFORMANCE_GUIDE.md     # Plugin documentation
└── .mcp.json                    # MCP config (if needed)
```

### Step 3: Register Plugin in marketplace.json

Add your plugin entry to `.claude-plugin/marketplace.json`:

```json
{
  "name": "deepakkamboj",
  "owner": {
    "name": "DevBuild Studio",
    "email": "support@devbuild.studio"
  },
  "plugins": [
    {
      "name": "accessibility",
      "source": "./plugins/accessibility",
      "description": "...",
      "version": "1.0.0",
      "author": {
        "name": "DevBuild Studio",
        "email": "support@devbuild.studio"
      },
      "strict": false
    },
    {
      "name": "performance",
      "source": "./plugins/performance",
      "description": "Performance optimization and monitoring for web applications",
      "version": "1.0.0",
      "author": {
        "name": "DevBuild Studio",
        "email": "support@devbuild.studio"
      },
      "strict": false
    }
  ]
}
```

### Step 4: Create Plugin README (Optional but Recommended)

Create a README for your plugin:

```bash
# plugins/performance/README.md
```

### Step 5: Update Main README

Add your plugin to the main [README.md](README.md) under a "Plugins" section:

```markdown
## Available Plugins

### Accessibility
A comprehensive accessibility-first development environment...
[Read more](plugins/accessibility/README.md)

### Performance
Performance optimization and monitoring for web applications...
[Read more](plugins/performance/README.md)
```

## Plugin Naming Conventions

- **Plugin directory name:** lowercase, hyphenated (e.g., `performance`, `security-scanner`)
- **Plugin name in marketplace.json:** Same as directory name
- **Agent files:** lowercase, hyphenated `.md` (e.g., `optimizer.md`, `security-auditor.md`)
- **Skill directories:** lowercase, hyphenated (e.g., `bundle-analyzer/`, `sql-injection-checker/`)
- **MCP servers (if any):** `mcp-server/` directory inside plugin

## User Installation

Users can install plugins in multiple ways:

### Install All Plugins from Marketplace

```bash
git clone https://github.com/deepakkamboj/accessibility.git
cd accessibility
claude plugin add ./
```

This installs ALL plugins defined in marketplace.json.

### Install Specific Plugin

```bash
git clone https://github.com/deepakkamboj/accessibility.git
cd accessibility
claude plugin add ./plugins/accessibility
claude plugin add ./plugins/performance
```

### Install Directly from GitHub (Single Plugin)

```bash
# If Claude supports GitHub-based installation
claude plugin add github:deepakkamboj/accessibility/plugins/accessibility
```

## Plugin Checklist

When adding a new plugin, ensure:

- [ ] Plugin directory created under `plugins/`
- [ ] At least one agent or skill defined
- [ ] Plugin registered in `.claude-plugin/marketplace.json`
- [ ] Plugin-specific documentation created
- [ ] Main README.md updated with plugin info
- [ ] If using MCP server:
  - [ ] `.mcp.json` configured
  - [ ] MCP server built (`npm run build`)
  - [ ] MCP tools documented
- [ ] Examples/usage scenarios provided
- [ ] Version number follows semantic versioning

## Shared Resources

Plugins can share resources across the repository:

- **Standards documents:** Place in root (e.g., `ACCESSIBILITY_STANDARDS.md`)
- **Common utilities:** Create `shared/` directory if needed
- **Documentation templates:** Create `templates/` directory

## MCP Server Guidelines

If your plugin includes an MCP server:

1. **Structure:**
   ```
   plugins/[plugin-name]/mcp-server/
   ├── src/
   │   └── index.ts
   ├── dist/
   ├── package.json
   └── README.md
   ```

2. **Package naming:** `@deepakkamboj/[plugin-name]-mcp`

3. **Binary name:** `[plugin-name]-mcp`

4. **Configuration in .mcp.json:**
   ```json
   {
     "mcpServers": {
       "plugin-name": {
         "command": "node",
         "args": ["${CLAUDE_PLUGIN_ROOT}/mcp-server/dist/index.js"]
       }
     }
   }
   ```

5. **Optional npm publication:**
   ```bash
   cd plugins/[plugin-name]/mcp-server
   npm publish --access public
   ```

## Example Plugin Ideas

Here are some plugin ideas for the DevBuild Studio marketplace:

- **Performance:** Bundle analysis, lighthouse audits, runtime profiling
- **Security:** OWASP checks, dependency scanning, code security analysis
- **Testing:** Test generation, coverage analysis, e2e testing
- **Documentation:** Auto-doc generation, API documentation, changelog creation
- **Code Quality:** Linting, formatting, complexity analysis
- **Internationalization:** i18n/l10n tools, translation management
- **Database:** Schema validation, query optimization, migration tools
- **API:** OpenAPI generation, REST/GraphQL validation, endpoint testing

## Versioning

Follow semantic versioning for each plugin:

- **Major (1.0.0):** Breaking changes to plugin API or structure
- **Minor (1.1.0):** New features, backward-compatible
- **Patch (1.0.1):** Bug fixes, backward-compatible

Update version in marketplace.json when releasing changes.

## Testing New Plugins

Before committing a new plugin:

1. **Test plugin installation:**
   ```bash
   claude plugin add ./plugins/[plugin-name]
   claude plugin list
   ```

2. **Test agents:**
   ```bash
   claude agent list [plugin-name]
   ```

3. **Test skills:**
   ```bash
   claude skill list [plugin-name]
   ```

4. **Test MCP server (if applicable):**
   ```bash
   cd plugins/[plugin-name]/mcp-server
   npm install
   npm run build
   node dist/index.js  # Should start without errors
   ```

## Questions?

Open an issue at https://github.com/deepakkamboj/accessibility/issues
