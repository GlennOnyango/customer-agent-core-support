# Customer Support Chatbot

## Project Overview

This project implements a customer-support chatbot with Amazon Bedrock Flow and an AgentCore managed harness. Customer messages are classified as bug reports, FAQ questions, or other requests and routed to separate response paths.

The bug-report workflow collects a description, steps to reproduce, and environment details before invoking a Lambda-backed tool through the AgentCore Gateway. Completed tickets are stored in DynamoDB. FAQ questions are answered from the included shop FAQ, while unsupported requests are directed to human support.

## Project Structure

```text
.
├── README.md
├── EVALUATION_OBSERVATION.md
├── system_prompt.txt
├── online_shop_faq.md
├── chat.py
├── create_harness.py
├── setup_gateway.py
├── create_bug_report.py
├── cleanup_agentcore.py
├── agentcore_config.json
├── harness-tests.json
├── harness-tests-template.json
├── generate-eval-dataset.py
├── output_eval_dataset.jsonl
├── cloudformation-tool.yaml
├── cloudformation-testing.yaml
├── requirements.txt
└── screenshots/
```

## Classification and Routing

The Bedrock Flow classifier routes customer messages across three distinct paths:

- `bug` for website or application malfunctions
- `faq` for questions covered by the shop FAQ
- A default path for unsupported or unrelated requests

Each path terminates at a separate Flow Output node.

Evidence:

- [Full flow diagram](<screenshots/full flow diagram screenshot.png>)
- [Classifier prompt configuration](<screenshots/classifier prompt configuration.png>)
- [Condition node expressions](<screenshots/Condition node expressions.png>)

## Bug Report Path

The collection rules for the bug-report route are defined in [system_prompt.txt](system_prompt.txt). The AgentCore managed harness uses the configured Gateway to invoke `bugreports___create_bug_report`. The Lambda implementation in [create_bug_report.py](create_bug_report.py) validates the required fields and writes completed tickets to the `bug-report-tool-stack-bug-reports` DynamoDB table.

Relevant implementation files:

- [chat.py](chat.py) provides the multi-turn command-line conversation.
- [create_harness.py](create_harness.py) creates or updates the managed harness.
- [setup_gateway.py](setup_gateway.py) configures the AgentCore Gateway and Lambda tool.
- [cloudformation-tool.yaml](cloudformation-tool.yaml) defines the Lambda, DynamoDB table, and supporting IAM resources.

Evidence:

- [Bug-report conversation](<screenshots/bug-report scenario.png>)
- [DynamoDB ticket records](<screenshots/dynamo-db.png>)

## Platform Questions and Other Requests

The FAQ content is stored in [online_shop_faq.md](online_shop_faq.md) and is inserted into the chatbot system prompt when the harness is created. Covered questions receive a concise answer based on the FAQ. Questions not covered by the FAQ and other requests are redirected to human support at `1-800-555-0199`, available Monday through Friday.

Evidence:

- [FAQ Prompt node](<screenshots/FAQ Prompt node.png>)
- [Covered FAQ question](<screenshots/Covered question.png>)
- [Uncovered question](<screenshots/Uncovered question.png>)
- [Other request](<screenshots/Other-request message.png>)

## Testing and Evaluation

[harness-tests.json](harness-tests.json) contains nine automated cases covering bug reports, FAQ questions, and other requests. [generate-eval-dataset.py](generate-eval-dataset.py) runs each test in a fresh harness session and produces [output_eval_dataset.jsonl](output_eval_dataset.jsonl) for a Bedrock LLM-as-a-judge evaluation.

The evaluation returned a correctness score of **1.00 across 9 prompts**.

Evidence:

- [Automated test suite](harness-tests.json)
- [Generated evaluation dataset](output_eval_dataset.jsonl)
- [Bedrock evaluation results](<screenshots/evaluation job.png>)
- [Evaluation observation](EVALUATION_OBSERVATION.md)

## Infrastructure and Setup Files

- [cloudformation-tool.yaml](cloudformation-tool.yaml) deploys the bug-report Lambda function, DynamoDB table, and IAM roles.
- [cloudformation-testing.yaml](cloudformation-testing.yaml) deploys the evaluation S3 bucket and Bedrock evaluation role.
- [requirements.txt](requirements.txt) lists the Python dependencies.
- [cleanup_agentcore.py](cleanup_agentcore.py) removes the AgentCore harness, Gateway target, and Gateway after testing.
