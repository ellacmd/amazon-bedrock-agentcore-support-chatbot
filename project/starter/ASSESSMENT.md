# Customer Support Chatbot — Assessment Evidence

## Implementation

Region: `us-east-1`

The chatbot runs in an Amazon Bedrock AgentCore managed harness using Amazon
Nova Pro (`us.amazon.nova-pro-v1:0`). Its system prompt routes every request to
one of three behaviors:

1. Bug reports collect a description, reproduction steps, and environment one
   question at a time, then call the Gateway tool.
2. Platform questions are answered only from `online_shop_faq.md`.
3. Other or unsupported requests are redirected to `1-800-555-0199 (Mon-Fri)`.

The prompt also treats user and FAQ text as untrusted input and redirects
prompt-injection attempts without revealing instructions or invoking tools.

Managed long-term harness memory is disabled in `create_harness.py`. Runtime
sessions still support multi-turn conversations, but facts cannot leak between
independent customers or automated test cases.

## Deployed resources

- Tool stack: `bug-report-tool-stack`
- DynamoDB table: `bug-report-tool-stack-bug-reports`
- Lambda: `bug-report-tool-stack-create-bug-report`
- Gateway: `bug-report-tool-stack-gateway-<generated-suffix>`
- Gateway target/tool: `bugreports___create_bug_report`
- Harness: `support_chatbot-<generated-suffix>`
- Harness ARN: `arn:aws:bedrock-agentcore:us-east-1:<AWS_ACCOUNT_ID>:harness/support_chatbot-<generated-suffix>`
- Testing stack: `bug-report-testing-stack`
- Evaluation bucket: `udacity-agentic-engineer-c1-eval-<AWS_ACCOUNT_ID>`
- Evaluation job: `support-chatbot-correctness-20260902-125405`
- Evaluation job ARN: `arn:aws:bedrock:us-east-1:<AWS_ACCOUNT_ID>:evaluation-job/<generated-id>`

`agentcore_config.json` contains the deployed harness and Gateway ARNs and is
included in this submission as requested by the reviewer.

## Verification results

- Direct Lambda invocation returned an `OPEN` ticket and persisted it in
  DynamoDB.
- A complete single-turn bug report invoked the Gateway tool and returned a
  real ticket ID.
- A three-turn bug conversation asked only for reproduction steps, then only
  for environment, then invoked `bugreports___create_bug_report` exactly once.
- A matching multi-turn ticket ID was verified in DynamoDB.
- `flow-tests.json` (also retained as `harness-tests.json`) covers FAQ answers, unsupported FAQ requests, unrelated
  requests, incomplete and complete bugs, an ambiguous payment question, very
  short input, and prompt injection.
- All nine final harness calls succeeded and produced valid JSONL in
  `output_eval_dataset.jsonl`.
- Dataset S3 URI:
  `s3://udacity-agentic-engineer-c1-eval-<AWS_ACCOUNT_ID>/input/output_eval_dataset.jsonl`
- Evaluation result prefix:
  `s3://udacity-agentic-engineer-c1-eval-<AWS_ACCOUNT_ID>/output/`

### Evaluation observation

Status: **Completed** with no failures.

Correctness result: **1.0 for all 9 of 9 test cases** (mean score **1.0**).
The judge confirmed correct FAQ grounding, phone redirection, one-question bug
intake, real ticket creation, ambiguous-payment handling, short-input handling,
and prompt-injection refusal.

Result file:
`s3://udacity-agentic-engineer-c1-eval-<AWS_ACCOUNT_ID>/output/<JOB_NAME>/<JOB_ID>/models/my-support-chatbot/taskTypes/General/datasets/support-chatbot-tests/<RESULT_ID>_output.jsonl`

During prompt refinement, the first harness version used managed long-term
memory by default. This contaminated fresh evaluation sessions with details
from earlier bug reports. Explicitly disabling managed memory fixed the issue
while preserving state within a single runtime session. The final prompt also
uses a fixed OTHER-route response template because free-form refusals sometimes
omitted the required phone number.

Amazon Nova Pro may emit a short `<thinking>` block immediately before a tool
call in verbose streaming output despite a system instruction not to expose
reasoning. The customer-facing final reply remains concise and includes the
real ticket ID.

## Commands to reproduce

From this directory, with the repository virtual environment already created:

```bash
../../.venv/bin/python setup_gateway.py
../../.venv/bin/python create_harness.py
../../.venv/bin/python chat.py
../../.venv/bin/python generate-eval-dataset.py --tests-json flow-tests.json
```

Verify tickets:

```bash
aws dynamodb scan \
  --table-name bug-report-tool-stack-bug-reports \
  --region us-east-1
```

## Screenshot checklist

The submitted screenshots and readable transcripts are indexed in
`evidence/README.md`.

Capture these in the AWS console or terminal for submission:

1. AgentCore Harness details showing `support_chatbot`, READY status, Nova Pro,
   and the system prompt.
2. AgentCore Gateway details showing READY status and the `bugreports` target
   with the `create_bug_report` tool schema.
3. Terminal `chat.py` conversation showing one-question-at-a-time bug intake,
   the tool call, and returned ticket ID.
4. DynamoDB table showing the matching ticket with description,
   `stepsToReproduce`, environment, and `OPEN` status.
5. Rendered `ARCHITECTURE.md` showing the complete prompt-defined route diagram.
6. Terminal or files showing `flow-tests.json` and the successful nine-case
   JSONL generation.
7. Bedrock Evaluations job details and correctness results after completion.

## Cleanup

Do not run these until the assessment evidence has been captured:

```bash
../../.venv/bin/python cleanup_agentcore.py
aws cloudformation delete-stack --stack-name bug-report-tool-stack --region us-east-1
aws s3 rm s3://udacity-agentic-engineer-c1-eval-<AWS_ACCOUNT_ID> --recursive
aws cloudformation delete-stack --stack-name bug-report-testing-stack --region us-east-1
```
