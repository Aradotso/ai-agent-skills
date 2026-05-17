```markdown
---
name: power-bi-agentic-development
description: Teach AI agents to build, manage, and optimize Power BI semantic models, reports, and Fabric artifacts using TMDL, PBIR, Tabular Editor, and CLI tools
triggers:
  - "help me with Power BI development"
  - "create a TMDL semantic model"
  - "validate my PBIP project structure"
  - "write a BPA rule for Tabular Editor"
  - "connect to Power BI Desktop and query the model"
  - "build a PBIR report definition"
  - "automate Fabric deployment"
  - "create DAX measures in TMDL format"
---

# power-bi-agentic-development

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

This project provides a comprehensive marketplace of plugins, skills, agents, and hooks that enable AI coding agents to work expertly with Power BI and Microsoft Fabric. It teaches agents to author TMDL semantic models, PBIR reports, C# scripts for Tabular Editor, Best Practice Analyzer rules, and automate workflows across the Power BI ecosystem.

## What This Project Does

- **Skills** teach agents domain knowledge for Power BI, TMDL, PBIR, Tabular Editor, DAX, and Fabric CLI
- **Agents** are autonomous subprocesses for validation, review, and complex multi-step tasks
- **Hooks** run automatically after file operations to validate TMDL syntax, PBIR schemas, DAX references, and metadata integrity
- **Plugins** bundle related skills, agents, and hooks for specific tools (Tabular Editor, PBIP, Power BI Desktop, Reports, Semantic Models, Fabric CLI)

## Installation

### Claude Code

Add the marketplace and install plugins:

```bash
claude plugin marketplace add data-goblin/power-bi-agentic-development
```

Then install via `/plugin` in the UI, or via CLI:

```bash
claude plugin install tabular-editor@power-bi-agentic-development
claude plugin install pbi-desktop@power-bi-agentic-development
claude plugin install semantic-models@power-bi-agentic-development
claude plugin install reports@power-bi-agentic-development
claude plugin install pbip@power-bi-agentic-development
claude plugin install fabric-cli@power-bi-agentic-development
```

Enable auto-update for the marketplace to get weekly releases automatically.

### Copilot CLI

```bash
copilot plugin marketplace add data-goblin/power-bi-agentic-development
copilot plugin install tabular-editor@power-bi-agentic-development
```

Or install a single plugin directly:

```bash
copilot plugin install data-goblin/power-bi-agentic-development:plugins/pbip
```

**Windows long path support**: TMDL files can exceed 260 characters. Enable long paths:

```powershell
# Run as Administrator
Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem' -Name LongPathsEnabled -Value 1
git config --system core.longpaths true
# Reboot recommended
```

## Key Plugins and Skills

### tabular-editor Plugin

#### BPA Rules Skill

Create and improve Best Practice Analyzer rules for semantic models.

```csharp
// Example BPA rule: Flag measures without descriptions
// File: rules/MeasuresMustHaveDescriptions.json
{
  "Name": "Measures must have descriptions",
  "Category": "Documentation",
  "Severity": 2,
  "Scope": "Measure",
  "Expression": "string.IsNullOrWhiteSpace(Description)",
  "FixExpression": "Description = \"[Auto-generated] TODO: Add description\";",
  "Description": "All measures should have a description for maintainability."
}
```

Command: `/suggest-rule` — Generate BPA rules from natural language descriptions.

#### C# Scripting Skill

Automate model changes with Tabular Editor C# scripts:

```csharp
// Add DisplayFolder and FormatString to all measures in a table
foreach(var measure in Model.Tables["Sales"].Measures)
{
    if(string.IsNullOrEmpty(measure.DisplayFolder))
    {
        measure.DisplayFolder = "Sales Metrics";
    }
    
    if(string.IsNullOrEmpty(measure.FormatString))
    {
        measure.FormatString = "#,0";
    }
}
```

#### Tabular Editor 2 CLI Skill

Automate TE2 via command line (not TE3):

```bash
# Deploy model with BPA validation
TabularEditor.exe "model.bim" `
  -SCRIPT "script.csx" `
  -DEPLOY "localhost" "AdventureWorks" `
  -BPA -BPAFILE "rules.json"

# Export to TMDL
TabularEditor.exe "model.bim" -FOLDER "output/tmdl"
```

### pbip Plugin

#### TMDL Skill

Author Tabular Model Definition Language files directly:

```tmdl
// File: model.tmdl
model Model
  culture: en-US
  defaultPowerBIDataSourceVersion: powerBI_V3

// File: tables/Sales.tmdl
table Sales
  lineageTag: abc123

  measure 'Total Sales' =
    SUM(Sales[SalesAmount])
    formatString: $#,0.00
    displayFolder: "Sales Metrics"
    description: "Sum of all sales amounts"

  column SalesAmount
    dataType: decimal
    sourceColumn: SalesAmount
    formatString: $#,0.00
    summarizeBy: sum
```

#### PBIR Format Skill

Author Power BI Report JSON files:

```json
// File: definition/report.json
{
  "config": "{\"version\":\"5.49\",\"themeCollection\":{\"baseTheme\":{\"name\":\"CY24SU09\"}}}",
  "layoutOptimization": 0,
  "resourcePackages": [
    {
      "resourcePackage": {
        "name": "SharedResources",
        "type": 1,
        "items": []
      }
    }
  ],
  "sections": [
    {
      "name": "ReportSection1",
      "displayName": "Sales Overview",
      "visualContainers": []
    }
  ],
  "publicCustomVisuals": []
}
```

#### PBIP Project Structure

```
MyReport.Report/
├── .platform
├── definition/
│   ├── report.json
│   └── pages/
│       └── ReportSection1.json
├── definition.pbir
└── item.config.json

MySemanticModel.SemanticModel/
├── .platform
├── definition/
│   ├── model.tmdl
│   ├── tables/
│   │   ├── Sales.tmdl
│   │   └── Customers.tmdl
│   └── relationships/
│       └── relationship-1.tmdl
└── definition.pbism
```

Hooks validate:
- PBIR schema compliance
- TMDL structural syntax
- Report binding to semantic models (byPath or byConnection)

Agent: `/agent pbip-validator` — Comprehensive PBIP validation.

### pbi-desktop Plugin

#### Connect to Power BI Desktop

Connect via XMLA endpoint (must enable in PBI Desktop settings):

```csharp
// C# example using Tabular Object Model
using Microsoft.AnalysisServices.Tabular;

var server = new Server();
server.Connect("localhost:12345"); // Port from XMLA endpoint

var database = server.Databases.GetByName("Model");
var model = database.Model;

// Query all measures
foreach(var table in model.Tables)
{
    foreach(var measure in table.Measures)
    {
        Console.WriteLine($"{table.Name}[{measure.Name}]: {measure.Expression}");
    }
}
```

#### Hooks for Connected Models

When connected to Power BI Desktop, hooks automatically:

1. **Validate DAX references** — Check table/column/measure names against live model
2. **Enforce measure metadata** — Block measures without DisplayFolder, Description, FormatString
3. **Check referential integrity** — Report unmatched keys after relationship changes
4. **Refresh metadata cache** — Auto-snapshot model schema on connect or modification
5. **Compatibility level check** — Suggest upgrades for new features

Configuration: `plugins/pbi-desktop/hooks/config.yaml`

```yaml
validate_dax_references: true
enforce_measure_metadata: true
check_referential_integrity: true
refresh_metadata_cache: true
check_compatibility_level: true
auto_upgrade_compatibility: false
```

### semantic-models Plugin

Work with published semantic models in Fabric:

```bash
# List models in workspace
fab list --workspace "Sales Workspace" --type SemanticModel

# Export model as TMDL
fab export --workspace "Sales Workspace" --item "Sales Model" --format TMDL --path ./export

# Deploy local TMDL to Fabric
fab deploy --workspace "Sales Workspace" --path ./model --format TMDL
```

### reports Plugin

#### Deneb Visuals Skill

Create Vega-Lite specifications for Deneb visuals:

```json
{
  "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
  "data": {"name": "dataset"},
  "mark": "bar",
  "encoding": {
    "x": {
      "field": "Category",
      "type": "nominal",
      "axis": {"labelAngle": -45}
    },
    "y": {
      "field": "Sales",
      "type": "quantitative",
      "aggregate": "sum"
    },
    "color": {
      "field": "Category",
      "type": "nominal",
      "legend": null
    }
  }
}
```

#### Report Theme Skill

Customize Power BI themes:

```json
{
  "name": "Corporate Theme",
  "dataColors": ["#003f5c", "#58508d", "#bc5090", "#ff6361", "#ffa600"],
  "background": "#FFFFFF",
  "foreground": "#333333",
  "tableAccent": "#003f5c",
  "textClasses": {
    "title": {
      "fontSize": 18,
      "fontFace": "Segoe UI Semibold",
      "color": "#003f5c"
    }
  }
}
```

### fabric-cli Plugin

Automate Fabric operations:

```bash
# Authenticate
fabric login

# Create workspace
fabric workspace create --name "Dev Environment"

# Deploy items
fabric deploy --workspace "Dev Environment" --path ./MyReport.Report

# Execute notebook
fabric notebook run --workspace "Analytics" --notebook "ETL Pipeline"

# Manage deployment pipelines
fabric pipeline deploy --pipeline "DTAP" --stage "TEST" --workspace "Dev Environment"
```

## Configuration

### Hook Configuration Files

Each plugin with hooks has a `hooks/config.yaml`:

**PBIP Plugin** (`plugins/pbip/hooks/config.yaml`):

```yaml
validate_pbir: true
validate_report_binding: true
validate_tmdl: true
```

**PBI Desktop Plugin** (`plugins/pbi-desktop/hooks/config.yaml`):

```yaml
validate_dax_references: true
enforce_measure_metadata: true
check_referential_integrity: true
refresh_metadata_cache: true
check_compatibility_level: true
auto_upgrade_compatibility: false
```

Set any check to `false` to disable it.

### Environment Variables

```bash
# Fabric CLI authentication
export FABRIC_TENANT_ID=your-tenant-id
export FABRIC_CLIENT_ID=your-client-id
export FABRIC_CLIENT_SECRET=your-client-secret

# Power BI Desktop XMLA endpoint
export PBI_XMLA_ENDPOINT=localhost:12345

# Tabular Editor paths
export TE2_PATH="C:\Program Files (x86)\Tabular Editor\TabularEditor.exe"
export TE3_PATH="C:\Program Files\Tabular Editor 3\TabularEditor3.exe"
```

## Common Patterns

### Create a New PBIP Project

```markdown
1. Create project structure:
   - MyReport.Report/ (report project)
   - MySemanticModel.SemanticModel/ (model project)

2. Author TMDL files in definition/ folder
   - model.tmdl (root)
   - tables/*.tmdl (one per table)
   - relationships/*.tmdl (one per relationship)

3. Author PBIR files in definition/ folder
   - report.json
   - pages/*.json

4. Hooks auto-validate on save
```

### Deploy to Fabric

```bash
# From PBIP project root
fabric deploy --workspace "Production" --path ./MySemanticModel.SemanticModel
fabric deploy --workspace "Production" --path ./MyReport.Report
```

### Run BPA Rules

```bash
# TE2 CLI
TabularEditor.exe "model.bim" -BPA -BPAFILE "rules/custom-rules.json"

# TE3 UI
# Load model → Tools → Best Practice Analyzer → Run
```

### Query Power BI Desktop Model

```csharp
// Connect via TOM
var server = new Server();
server.Connect("localhost:12345");
var model = server.Databases[0].Model;

// List all measures with format strings
foreach(var table in model.Tables)
{
    foreach(var measure in table.Measures)
    {
        Console.WriteLine($"{measure.Name}: {measure.FormatString}");
    }
}
```

## Troubleshooting

### "Filename too long" on Windows

Enable long path support (see Installation section). Reboot after registry change.

### PBIR Validation Fails

Check schema URLs match Power BI version:

```json
"$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/visualContainer/1.2.0/schema.json"
```

### TMDL Syntax Errors

Hooks flag common errors:
- Missing lineageTag
- Invalid data types
- Malformed DAX expressions

Run `/agent pbip-validator` for detailed analysis.

### Hooks Not Firing in Copilot CLI

Requires Copilot CLI ≥ 1.0.26 (2026-04-14). Update:

```bash
copilot update
```

### DAX Reference Validation Fails

Ensure metadata cache is current:

```bash
# Refresh cache manually
/skill connect-pbid refresh-metadata
```

Or set `refresh_metadata_cache: true` in config.

### BPA Rule Expression Errors

Use `/agent bpa-expression-helper` to debug and improve rule expressions. Common issues:
- Missing null checks
- Incorrect scope (Model vs. Table vs. Measure)
- Invalid C# syntax in expressions

## Skills Reference

| Plugin | Skill | Description |
|--------|-------|-------------|
| tabular-editor | `bpa-rules` | Create BPA rules |
| tabular-editor | `c-sharp-scripting` | TE C# scripts and macros |
| tabular-editor | `te2-cli` | TE2 CLI automation |
| tabular-editor | `te-docs` | Search TE documentation |
| pbi-desktop | `connect-pbid` | Connect to PBI Desktop via XMLA |
| pbip | `pbip` | PBIP project format |
| pbip | `tmdl` | Author TMDL files |
| pbip | `pbir-format` | Author PBIR JSON |
| reports | `deneb-visuals` | Vega/Vega-Lite specs |
| reports | `modifying-theme-json` | Theme files |
| reports | `pbi-report-design` | Report best practices |
| semantic-models | `semantic-models` | Published model operations |
| fabric-cli | `fabric-cli` | Fabric CLI automation |

## Agents Reference

| Agent | Plugin | Purpose |
|-------|--------|---------|
| `pbip-validator` | pbip | Validate PBIP structure and schemas |
| `bpa-expression-helper` | tabular-editor | Debug BPA rule expressions |
| `query-listener` | pbi-desktop | Capture DAX queries from visuals |

## Commands Reference

| Command | Plugin | Description |
|---------|--------|-------------|
| `/suggest-rule` | tabular-editor | Generate BPA rule from description |

## Version and Updates

Current version: **26.20** (weekly release cadence)

Enable marketplace auto-update in Claude Code to receive new skills, agents, and bug fixes automatically.

```bash
# Manual update
claude plugin marketplace update power-bi-agentic-development
```

## License

GPL-3.0 — See repository for full license text.

## Additional Resources

- [Tabular Editor Documentation](https://docs.tabulareditor.com)
- [TMDL Specification](https://learn.microsoft.com/power-bi/developer/projects/projects-dataset)
- [PBIR Schema Reference](https://learn.microsoft.com/power-bi/developer/projects/projects-report)
- [Fabric CLI Documentation](https://learn.microsoft.com/fabric/cicd/deployment-pipelines/pipeline-automation)
- [pbi-search CLI](https://github.com/data-goblin/pbi-search) — Documentation search tool used by te-docs skill
```
