# Aleutian Client (Python SDK)

[![PyPI version](https://badge.fury.io/py/aleutian-client.svg)](https://badge.fury.io/py/aleutian-client)
[![License: AGPL v3](https://img.shields.io/badge/License-AGPL%20v3-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)

The official Python client SDK for the **[AleutianLocal Secure Enterprise Intelligence Platform](https://github.com/jinterlante1206/AleutianLocal)**.

This package allows developers, data scientists, and applications to programmatically interact with a running AleutianLocal stack to perform:
* **Autonomous Agent Tasks:** Deploy the local agent to trace code and answer architectural questions.
* **RAG Queries:** Ask questions against your private data with full citations.
* **Direct Chat:** Interact with the configured LLM (Claude, OpenAI, or Local) with support for "Thinking Mode".
* **Timeseries Forecasting:** Run forecasts on financial or custom data.

---

## Prerequisites

**This package is a client library.** It **requires** the full **[AleutianLocal](https://github.com/jinterlante1206/AleutianLocal)** stack to be installed and running on your local machine.

Before using this package, please ensure you have:
1.  Installed the `aleutian` CLI (see the [main project for instructions](https://github.com/jinterlante1206/AleutianLocal#installation)).
2.  Started the Aleutian stack:

    ```bash
    aleutian stack start
    ```

The `AleutianClient` will connect to the `orchestrator` service, which runs on `http://localhost:12210` by default.

## Installation

You can install the client using `pip`:

```bash
pip install aleutian-client
```

## Quickstart & Usage
The client is designed to be used with a context manager (the with statement), which automatically handles opening and closing the connection.

```Python
import sys
from aleutian_client import AleutianClient, Message
from aleutian_client import AleutianConnectionError, AleutianApiError

def main():
    try:
        # 1. Connect to the running Aleutian stack
        # Defaults to host="http://localhost", port=12210
        with AleutianClient() as client:

            # 2. Run a health check to verify connection
            health = client.health_check()
            print(f"✅ Successfully connected: {health.get('status')}")

            # -------------------------------------------------
            # Example 1: Autonomous Agent (New in v0.3.0)
            # Deploy the agent to analyze code or answer complex questions
            # -------------------------------------------------
            print("\n--- 1. Agent Trace ---")
            try:
                # The agent will explore the codebase available to the stack
                trace_resp = client.trace(query="Analyze the auth logic in cmd_stack.go")
                print(f"Agent Answer: {trace_resp.answer}")
                
                if trace_resp.steps:
                    print("Steps Taken:")
                    for step in trace_resp.steps:
                        # step is an AgentStep object
                        print(f" - {step.tool}({step.args})")
            except AleutianApiError as e:
                print(f"Agent Error: {e}")

            # -------------------------------------------------
            # Example 2: RAG-Powered Ask
            # -------------------------------------------------
            print("\n--- 2. RAG-Powered Query ---")
            try:
                response_rag = client.ask(
                    query="What is AleutianLocal?",
                    pipeline="reranking" # or "standard"
                )
                print(f"RAG Answer: {response_rag.answer}")
                
                if response_rag.sources:
                    print(f"Sources: {[s.source for s in response_rag.sources]}")
            
            except AleutianApiError as e:
                print(f"API Error: {e}")

            # -------------------------------------------------
            # Example 3: Direct Chat with Thinking (Claude 3.7+)
            # -------------------------------------------------
            print("\n--- 3. Direct Chat (Thinking Mode) ---")
            try:
                messages = [
                    Message(role="user", content="Explain the implications of P=NP.")
                ]
                # Enable extended thinking for complex tasks
                # Note: Requires a backend that supports thinking (e.g., Claude 3.7)
                response_chat = client.chat(
                    messages=messages,
                    enable_thinking=True,
                    budget_tokens=4000
                )
                print(f"Chat Answer: {response_chat.answer}")
            
            except AleutianApiError as e:
                print(f"API Error: {e}")
            
            # -------------------------------------------------
            # Example 4: Timeseries Forecasting (Experimental)
            # -------------------------------------------------
            print("\n--- 4. Forecasting ---")
            try:
                # Requires 'timeseries' feature enabled in Aleutian config
                forecast = client.forecast(
                    name="SPY",
                    context_period_size=300,
                    forecast_period_size=20
                )
                print(f"Forecast generated: {len(forecast.forecast)} points")
            except AleutianApiError as e:
                print(f"Forecast Error: {e}")

    except AleutianConnectionError:
        print("\nError: Could not connect to AleutianLocal stack.", file=sys.stderr)
        print("Please ensure the stack is running with 'aleutian stack start'.", file=sys.stderr)
        sys.exit(1)
    except Exception as e:
        print(f"An unexpected error occurred: {e}", file=sys.stderr)
        sys.exit(1)

if __name__ == "__main__":
    main()
```

## License

This project is licensed under the GNU Affero General Public License v3.0.
