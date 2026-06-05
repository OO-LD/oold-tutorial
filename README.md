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

!Note: The `@context` and `$schema` keywords are used in the instance documents to reference the same schema file, enabling both JSON Schema validation and JSON-LD processing without any additional transformation.
- The `$schema` keyword is used to declare which dialect of JSON Schema the schema was written for. The value of the 
  `$schema` keyword is also the identifier for a schema that can be used to verify that the schema is valid according to the dialect $schema identifies. A schema that describes another schema is called a "meta-schema". [Definition of the $schema keyword](https://json-schema.
  org/understanding-json-schema/reference/schema#schema)
- [Info on the @context keyword](https://www.w3.org/TR/2020/REC-json-ld11-20200716/#the-context)

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
!Note: relative reference in jane-smith.json to PersonWithAddress.schema.json will be resolved by the validator. Works only if CWD is set to examples/ and the file is present in the directory.

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

!Note: If one was to select the PersonWithAddress.schema.json, the code generator would fail because Address.schema.json is messing. 

## Key Concepts Demonstrated

### Bidirectional Referencing

The instance document `john-doe.json` references `Person.schema.json` in two ways:
- `"@context": "Person.schema.json"` - for JSON-LD processing
- `"$schema": "Person.schema.json"` - for JSON Schema validation

Both point to the **same file**, which is the core innovation of OO-LD.

!Note: John-doe.json is referencing the Person.schema.json file, which is in the same directory. The reference 
keyword `$schema` is used to indicate that the instance document should be validated against the schema defined in `Person.schema.json`. 

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

!Note: Is there any way to shorten this? Can the PersonWithAddress.schema.json file reference the Address.schema.
json file only once? If not, we need an explanation of why this is the case:
- The reason for the dual reference in `PersonWithAddress.schema.json` is that JSON Schema and JSON-LD have different mechanisms for referencing external definitions. JSON Schema uses `$ref` to include external schemas for validation purposes, while JSON-LD uses `@context` to define semantic mappings. Since these are separate concerns, both references are necessary to ensure that the schema functions correctly in both contexts.
