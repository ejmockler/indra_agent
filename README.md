# INDRA Agent

MCP server and agent tools for INDRA CoGEx knowledge graph exploration.

Enables AI agents to explore biomedical knowledge through the Model Context Protocol (MCP). Replaces 100+ individual function tools with 9 composable tools that expose the full power of Cypher queries while maintaining safety and usability.

## Installation

```bash
pip install indra_agent
```

Or from source:

```bash
git clone https://github.com/gyorilab/indra_agent.git
cd indra_agent
pip install -e ".[dev]"
```

## Configuration

Set Neo4j credentials via environment variables:

```bash
export INDRA_NEO4J_URL="bolt://localhost:7687"
export INDRA_NEO4J_USER="neo4j"
export INDRA_NEO4J_PASSWORD="your-password"
```

Or create a `.env` file in your working directory.

## Usage

### Run the MCP Server

```bash
# Via console script
indra-agent

# Or via module
python -m indra_agent.mcp_server
```

### Claude Desktop Configuration

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "indra-cogex": {
      "command": "indra-agent",
      "env": {
        "INDRA_NEO4J_URL": "bolt://your-server:7687",
        "INDRA_NEO4J_USER": "neo4j",
        "INDRA_NEO4J_PASSWORD": "your-password"
      }
    }
  }
}
```

## Tools

**9 tools** organized into two groups:

### Gateway Tools (4 tools)
High-level graph navigation—most agents start here:

| Tool | Purpose |
|------|---------|
| `ground_entity` | Natural language → CURIE with semantic filtering |
| `suggest_endpoints` | Given CURIEs, suggest reachable entity types and functions |
| `call_endpoint` | Execute any of 190+ autoclient functions with auto-grounding |
| `get_navigation_schema` | Full edge map showing how entity types connect |

### Query Infrastructure (5 tools)
Low-level Cypher access for complex queries:

| Tool | Purpose |
|------|---------|
| `get_graph_schema` | Discover entity types, relationships, patterns |
| `execute_cypher` | Run arbitrary Cypher with parameterization |
| `validate_cypher` | Pre-flight safety validation |
| `enrich_results` | Add metadata at configurable disclosure levels |

## Architecture

Most agents use Gateway Tools—ground natural language to CURIEs, then call pre-built functions. When predefined functions cannot express the query (graph algorithms, multi-hop traversals, conditional aggregations), agents drop down to Query Infrastructure: discover schema, execute Cypher with validation, enrich results.

### Context-Aware Grounding

Parameter semantics encode entity type. When `call_endpoint` receives `disease="ALS"`, it filters GILDA results to disease namespaces:

```python
ground_entity(term="ALS", param_name="disease")
# Returns: MESH:D000690 (Amyotrophic Lateral Sclerosis)

ground_entity(term="ALS", param_name="gene")
# Returns: HGNC:396 (SOD1, formerly ALS1)
```

### Safety

- **Validation layer** prevents destructive operations (DELETE, CREATE, MERGE)
- **Parameterized queries** prevent injection attacks
- **Neo4j `execute_read()`** enforces read-only semantics at the driver level

### Token-Aware Pagination

Large result sets are automatically truncated with continuation hints:

```python
{
  "results": [...],
  "pagination": {
    "total": 1500,
    "returned": 127,
    "has_more": true,
    "next_offset": 127
  },
  "continuation_hint": "To get more, call with offset=127"
}
```

## Development

```bash
# Install with dev dependencies
pip install -e ".[dev]"

# Run tests
pytest tests/

# Run specific test file
pytest tests/mcp_server/test_gateway_tools.py -v
```

## Dependencies

- `indra_cogex` - INDRA CoGEx knowledge graph client
- `mcp>=1.2.0` - Model Context Protocol SDK
- `gilda` - Biomedical entity grounding
- `pydantic>=2.0` - Data validation

## License

BSD-2-Clause
