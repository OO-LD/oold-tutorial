# OO-LD Tutorial

This repository contains minimal working examples demonstrating OO-LD concepts.

## Files

### Basic Example
- **examples/Person.schema.json**: A minimal OO-LD schema defining a Person with a name property
- **examples/john-doe.json**: An instance document conforming to the Person schema

### Composition Example
- **examples/Address.schema.json**: An OO-LD schema defining an Address
- **examples/PersonWithAddress.schema.json**: A composed schema referencing Address
- **examples/jane-smith.json**: An instance with nested address object

## What Makes This OO-LD?

The `Person.schema.json` file is simultaneously:
1. A valid **JSON Schema** (validates instance documents)
2. A **JSON-LD remote context** (provides semantic mappings)

Notice how the same file contains both:
- JSON Schema structure (`type`, `properties`, `description`)
- JSON-LD semantics (`@context` mapping `name` to `schema:name`)

> [!NOTE]
> Both `$schema` and `@context` in an instance document can point to the same OO-LD schema file, enabling JSON Schema validation and JSON-LD processing without any additional transformation. 
> See the OO-LD spec's [Schema Instances](https://oo-ld.github.io/oold-schema/latest/spec/#schema-instances) section, and the [`$schema` keyword](https://json-schema.org/understanding-json-schema/reference/schema#schema) and [`@context` keyword](https://www.w3.org/TR/2020/REC-json-ld11-20200716/#the-context) docs for details.

## Try It Yourself

### Prerequisites

Install `uv`:

```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### Setup

```bash
git clone https://github.com/OO-LD/oold-tutorial
cd oold-tutorial
uv sync
cd examples
```

### 1. Validate with JSON Schema

```bash
# Validate the basic example
uv run check-json-schema-meta john-doe.json
```

Output:
```
✅ john-doe.json: Schema validation passed
```

```bash
# Validate the composition example
uv run check-json-schema-meta jane-smith.json
```
> [!IMPORTANT]
> Schema references are resolved relative to the current working directory. Run these commands from `examples/`.

Output:
```
✅ jane-smith.json: Schema validation passed
```

### 2. Generate RDF

```bash
# Generate RDF from basic example
uv run rdfpipe john-doe.json
```

Output (Turtle format):
```turtle
@prefix schema1: <http://schema.org/> .

[] schema1:name "John Doe" .
```

```bash
# Generate RDF from composition example
uv run rdfpipe jane-smith.json
```

Output (Turtle format):
```turtle
@prefix schema1: <http://schema.org/> .

[] schema1:address [ schema1:addressLocality "Springfield" ;
            schema1:postalCode "12345" ;
            schema1:streetAddress "123 Main Street" ] ;
    schema1:email "jane.smith@example.com" ;
    schema1:name "Jane Smith" .
```

### 3. Interactive Playground

Copy the content of `examples/Person.schema.json` into the [OO-LD Playground](https://oo-ld.github.io/playground-yaml/) to:
- See the schema and instance side-by-side
- Generate a web form automatically
- View the RDF output in real-time

### 4. Generate Python Code

Try code generation in the [Python Playground](https://oo-ld.github.io/playground-python-yaml/):
- Paste your OO-LD schema
- See generated Pydantic dataclasses with embedded `@context`
- Test round-trip conversion between schemas and Python code

> [!WARNING]
> The Python Playground cannot resolve external file references. Pasting `PersonWithAddress.schema.json` will fail because `Address.schema.json` is not available remotely — use `Person.schema.json` instead.

## Key Concepts Demonstrated

### Bidirectional Referencing

The instance document `john-doe.json` references `Person.schema.json` in two ways:
- `"@context": "Person.schema.json"` - for JSON-LD processing
- `"$schema": "Person.schema.json"` - for JSON Schema validation

Both point to the **same file**, which is the core innovation of OO-LD.

### No Processing Required

The `Person.schema.json` file needs **no transformation** to work as:
- A JSON Schema validator input
- A JSON-LD remote context

Standard tools work directly with the file as-is.

### Schema Composition

The `PersonWithAddress.schema.json` demonstrates how OO-LD handles composition:

**JSON Schema side (`$ref`):**
```json
"address": {
  "type": "object",
  "$ref": "Address.schema.json"
}
```

**JSON-LD side (scoped context):**
```json
"address": {
  "@id": "schema:address",
  "@context": "Address.schema.json"
}
```

Both reference the **same file** (`Address.schema.json`), keeping schema inheritance and semantic context synchronized automatically. This is the key difference from approaches that maintain separate JSON Schema and JSON-LD context files.

When you validate `jane-smith.json`:
- JSON Schema validates the nested address structure
- JSON-LD generates proper RDF with `schema:address` and nested address properties

> [!NOTE]
> `PersonWithAddress.schema.json` references `Address.schema.json` twice because JSON Schema (`$ref`) and JSON-LD (`@context`) are separate processing mechanisms with no shared reference syntax — both are required for the schema to work correctly in both contexts. See [Composition](https://oo-ld.github.io/oold-schema/latest/spec/#composition) in the OO-LD spec for the reflection rule.
