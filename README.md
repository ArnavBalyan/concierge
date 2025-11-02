<br>
<p align="center">
  <img src="assets/logo.svg" alt="Concierge Logo" width="2000"/>
</p>
<br>

<h3 align="center">
A declarative framework for exposing services to AI agents
</h3>

<p align="center">
| <a href="#about"><b>About</b></a> | <a href="#getting-started"><b>Getting Started</b></a> | <a href="#examples"><b>Examples</b></a> | <a href="#contributing"><b>Contributing</b></a> |
</p>

---

[![Build Status](https://img.shields.io/badge/build-passing-brightgreen)]() [![License: MIT](https://img.shields.io/badge/License-MIT-blue)]() [![Python 3.10+](https://img.shields.io/badge/python-3.10+-lightgrey)]()

## About

Concierge is a server-centric framework for building reliable AI agent workflows. Business logic stays server-side; agents interact through a language engine that handles validation, parameter collection, and state management.

Built for high-stakes agentic interactions where domain-specific logic must be abstracted from agents and enforced server-side.

Concierge provides:

- **Declarative workflows**: Define stages and tasks using Python decorators
- **Automatic validation**: Type-safe parameters with auto-prompting for missing values
- **State management**: Session state persisted across tasks and stages
- **Controlled transitions**: Valid paths between stages enforced by framework
- **Prerequisites enforcement**: Required state fields validated before stage entry
- **Language engine**: Translates workflows into agent-understandable prompts

## Features

**For Developers:**
- Type-safe task definitions with automatic parameter validation
- Decorator-based workflow definition
- Session management with state persistence
- Multi-stage workflows with controlled transitions
- REST API server included

**For Production:**
- Parameter validation before task execution
- Stage prerequisites enforcement
- Error handling and graceful failures
- Audit trail of agent actions
- Business rules enforced server-side

## Getting Started

Install Concierge:

```bash
pip install -e .
```

Run the server:

```bash
./scripts/run-server.sh
```

Define a workflow:

```python
from concierge.core import workflow, stage, task, State

@stage(name="processing")
class ProcessingStage:
    @task(description="Process data with validation")
    def process(self, state: State, input_data: str) -> dict:
        result = self.validate_and_process(input_data)
        state.set("result", result)
        return {"status": "success", "result": result}

@workflow(name="data_processor")
class DataWorkflow:
    processing = ProcessingStage
    transitions = {}
```

Connect an agent:

```bash
curl -X POST http://localhost:8082/execute \
  -H "Content-Type: application/json" \
  -d '{"action": "handshake", "workflow_name": "data_processor"}'
```

## Examples

### Multi-Stage Workflow

```python
@workflow(name="amazon_shopping")
class AmazonShoppingWorkflow:
    browse = BrowseStage         # Search and filter products
    select = SelectStage         # Add items to cart
    checkout = CheckoutStage     # Complete transaction
    
    transitions = {
        browse: [select],
        select: [browse, checkout],
        checkout: []
    }
```

### Stage with Tasks

```python
@stage(name="browse")
class BrowseStage:
    @task(description="Search for products by keyword")
    def search_products(self, state: State, query: str) -> dict:
        """Returns matching products"""
        
    @task(description="Filter products by price range")
    def filter_by_price(self, state: State, min_price: float, max_price: float) -> dict:
        """Filters current results by price"""
        
    @task(description="Sort products by rating or price")
    def sort_products(self, state: State, sort_by: str) -> dict:
        """Sorts: 'rating', 'price_low', 'price_high'"""

@stage(name="select")
class SelectStage:
    @task(description="Add product to shopping cart")
    def add_to_cart(self, state: State, product_id: str, quantity: int) -> dict:
        """Adds item to cart"""
        
    @task(description="Save product to wishlist")
    def add_to_wishlist(self, state: State, product_id: str) -> dict:
        """Saves item for later"""
        
    @task(description="Star product for quick access")
    def star_product(self, state: State, product_id: str) -> dict:
        """Stars item as favorite"""
        
    @task(description="View product details")
    def view_details(self, state: State, product_id: str) -> dict:
        """Shows full product information"""
```

### Prerequisites

```python
@stage(name="checkout", prerequisites=["cart.items", "user.payment_method"])
class CheckoutStage:
    @task(description="Apply discount code")
    def apply_discount(self, state: State, code: str) -> dict:
        """Validates and applies discount"""
        
    @task(description="Complete purchase")
    def complete_purchase(self, state: State) -> dict:
        """Processes payment and creates order"""
```

## Use Cases

- **Data Debugging**: Navigate Spark logs, traces, and metrics through structured stages
- **Financial Services**: Payment workflows with compliance checks and audit trails
- **Infrastructure Operations**: Deploy services with validation gates and rollback logic
- **Customer Support**: Escalation workflows with context preservation

## Architecture

```
┌─────────────┐
│    Agent    │  Natural language request
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────────┐
│      Concierge Language Engine      │
│  • Parses intent                    │
│  • Validates parameters             │
│  • Checks prerequisites             │
│  • Generates prompts                │
└──────┬──────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────┐
│         Your Workflow               │
│  @task                              │
│  def process(state, params)         │
│      # Your business logic          │
└─────────────────────────────────────┘
```

## Contributing

Contributions are welcome. Please open an issue or submit a pull request.

## License

MIT
