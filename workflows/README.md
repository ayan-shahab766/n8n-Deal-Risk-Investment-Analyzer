# n8n Deal Risk & Investment Analyzer workflow import map

Import the workflow JSON files with every workflow **inactive**. n8n workflow IDs are instance-specific, so the Execute Workflow nodes in the original orchestrator must be remapped after import.

## Deal Risk sequence

```text
WF0 orchestrator
  → WF#1 Deal Risk Intake & Normalize
  → WF#2 Enrich (Census)
  → WF#3 Math Engine
  → WF#6 Scenario & Risk Engine
  → WF#4 AI Strategy
  → WF#7 Human Review Gate
  → WF#5 Notion Writer
```

## Required remapping

1. Import WF1–WF7 and then WF0.
2. In WF0, open every **Execute Workflow** node and select the corresponding workflow by name in your n8n instance. Do not reuse the exported IDs.
3. Insert the two new stages after **Call WF3**: `Scenario & Risk Engine`, then `AI Strategy`, then `Human Review Gate`.
4. Map the WF7 fields to Notion: `reviewStatus`, `recommendation`, `riskRegister`, `scenarios`, `idempotencyKey`, and `auditTrail`.
5. Add a Notion lookup (or Data Store lookup) using `idempotencyKey` before creating a record. If a deal already exists, update it rather than creating a duplicate.
6. Protect the WF0 webhook with Header Auth and keep it inactive until all credentials and mappings are tested.

The review gate deliberately prepares a queue payload. It does not approve/reject a property, contact a seller, or make an investment decision.
