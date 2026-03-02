# MCPF Schema for Auto-Complete Feature

## Features
- Only pipeline elements defined in the schema are supported.  
- Autocomplete is available only for known built-in imports, though arbitrary modules are still allowed.  
- Known built-in pipeline functions include autocomplete and validation:
  - Their parameters are autocompleted and validated as well.  
- Any other pipeline function names are allowed as a generic fallback. 

## Notes / Limitations
- No autocomplete suggestions for custom modules and functions — only built-in steps appear
- Dynamic validation (e.g., loop: referencing a pipeline) is not supported.
- Parameters are validated strictly only for built-in steps defined in the schema.

---

## How to Use It in PyCharm

Save this schema file as `mcpf.schema.json`.

1. Open **PyCharm → Settings → Languages & Frameworks → Schemas and DTDs → JSON Schema**.  
2. Add the schema and select this file.  
3. Choose a mapping option:

### By File Pattern 

- Use the pattern: `*.mcpf*.yaml`  

### By Content Marker (Not available in community edition)

- Add `#mcpf` as the first line in the YAML file and use a PyCharm version supporting **“Schema from content detection”**.

---

## How to Use It in VSCode

### Prerequisites

- Install the **YAML extension**.  
- Recommended: **Red Hat YAML** extension (by Red Hat).  
- Ensure the JSON Schema file (`mcpf.schema.json`) is accessible locally or via HTTP(S).

### Schema Association

VSCode supports associating schemas either by **file pattern** or **in-file comment**.

---

### Option 1 — File Pattern Mapping

Add the following to your VSCode `settings.json`:

```json
"yaml.schemas": {
    "file:///absolute/path/to/mcpf.schema.json": "*.mcpf*.yaml"
}
```

---

### Option 2 — `$schema` Comment in YAML (Content-based)

Insert at the top of your YAML file:

```yaml
# yaml-language-server: $schema=file:///absolute/path/to/mcpf.schema.json

```

