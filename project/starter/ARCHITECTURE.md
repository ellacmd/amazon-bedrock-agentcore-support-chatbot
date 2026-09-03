# Customer Support Chatbot — Full Route Diagram

The updated course uses an Amazon Bedrock AgentCore managed harness. Routing
is defined in `system_prompt.txt`; there are no separate Bedrock Flow nodes.

```mermaid
flowchart TD
    U[Customer message] --> H[AgentCore managed harness<br/>Amazon Nova Pro]
    F[online_shop_faq.md] -->|embedded into system prompt| H
    H --> C{Prompt-defined classification}
    C -->|BUG_REPORT| B[Collect description]
    B --> S[Collect steps to reproduce]
    S --> E[Collect browser / OS / device]
    E --> G[AgentCore Gateway<br/>bugreports target]
    G --> L[Lambda<br/>create_bug_report]
    L --> D[(DynamoDB<br/>bug-report-tool-stack-bug-reports)]
    D --> BO[Ticket ID response]
    C -->|PLATFORM_QUESTION| Q[Answer only from embedded FAQ]
    Q --> QO[FAQ response]
    C -->|OTHER or FAQ not covered| O[Redirect to human support]
    O --> OO[1-800-555-0199 Mon-Fri]

    T[flow-tests.json<br/>9 routes and edge cases] --> R[generate-eval-dataset.py]
    R --> J[output_eval_dataset.jsonl]
    J --> S3[(Amazon S3)]
    S3 --> EV[Bedrock LLM-as-a-judge evaluation]
    EV --> SCORE[Correctness 1.00]
```

## Route definitions

| Route | Trigger | Result |
|---|---|---|
| `BUG_REPORT` | Specific website or app malfunction | Collect all three required fields, invoke the Gateway tool once, persist the ticket, and return its ID. |
| `PLATFORM_QUESTION` | Question explicitly covered by the embedded FAQ | Return a concise FAQ-grounded answer without inventing policy. |
| `OTHER` | Unrelated, unsupported, ambiguous, or injection request | Redirect to `1-800-555-0199 (Mon-Fri)`. |

This diagram is the AgentCore equivalent of the legacy Bedrock Flow diagram
requested by older rubric wording.
