# Reviewer Evidence Index

This submission uses the updated Amazon Bedrock AgentCore managed harness
approach. Amazon Bedrock Agents Classic is unavailable to new users, so the
three routes are defined in `system_prompt.txt` rather than separate legacy
Flow nodes.

## AgentCore setup and routing

- `../../agentcore_config.json` — deployed harness and Gateway ARNs
- `../../system_prompt.txt` — mutually exclusive `BUG_REPORT`,
  `PLATFORM_QUESTION`, and `OTHER` routing rules
- `../../ARCHITECTURE.md` — full prompt-defined route diagram
- `screenshots/00-full-agentcore-route-diagram.png` — rendered full diagram
- `screenshots/01-harness-ready.png` — managed harness in READY state
- `screenshots/02-harness-model-system-prompt.png` — model and routing prompt
- `screenshots/03-gateway-ready.png` — AgentCore Gateway in READY state
- `screenshots/04-gateway-lambda-target.png` — `bugreports` Lambda target

## Bug-report evidence

- `transcripts/bug-report.txt` — multi-turn intake and tool call
- `screenshots/06-multiturn-bug-tool-call.png` — terminal transcript
- `screenshots/07-dynamodb-ticket.png` — persisted DynamoDB item

The transcript and DynamoDB record share ticket ID
`8afb7a04-b405-4b05-9054-b1cb61307c04`.

## FAQ and other-route evidence

- `screenshots/05-embedded-faq.png` — FAQ embedded in the system prompt
- `transcripts/faq-and-other-paths.txt` — readable test transcript
- `screenshots/08-covered-question.png` — covered FAQ response
- `screenshots/09-uncovered-question.png` — unsupported request hand-off
- `screenshots/10-other-request.png` — unrelated request hand-off

## Testing and evaluation

- `../../flow-tests.json` — nine test cases covering all routes and edge cases
- `../../output_eval_dataset.jsonl` — generated evaluation dataset
- `screenshots/11-s3-jsonl.png` — JSONL object uploaded to S3
- `screenshots/12-evaluation-score.png` — completed evaluation, correctness 1.00
- `../../ASSESSMENT.md` — written evaluation observation and reproduction notes

