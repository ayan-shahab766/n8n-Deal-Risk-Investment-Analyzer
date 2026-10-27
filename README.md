# n8n Deal Risk & Investment Analyzer

n8n Deal Risk & Investment Analyzer is a modular real-estate underwriting workflow that transforms a property lead into an auditable acquisition-review package. It combines deterministic investment math, market enrichment, scenario analysis, AI-assisted strategy, and a required human review gate.

## What is new in Deal Risk

- **Strategy-aware underwriting** for fix-and-flip, long-term rental, BRRRR, and wholesale leads.
- **Stable idempotency fingerprints** from normalized address, asking price, and strategy mode to support duplicate-safe intake.
- **Downside, base, and upside scenarios** with ARV, repair, rent, cap-rate, DSCR, and equity-spread sensitivity.
- **Explainable risk register** for weak ARV evidence, high rehab burden, missing market rent, low DSCR, downside losses, and incomplete market context.
- **Human review gate** that generates an auditable queue payload and explicitly does not make an automated purchase, rejection, or seller-contact decision.
- **Separation of deterministic math and AI**: formulas and risk rules are inspectable JavaScript; Gemini only assists with the narrative resale strategy.

## Architecture

```text
Lead webhook / form
  → Intake, validation & idempotency key
  → Census enrichment
  → deterministic core underwriting math
  → downside / base / upside scenario engine
  → AI acquisition strategy
  → human review queue payload
  → Notion deal record + webhook confirmation
```

## Workflows

| Workflow | Purpose |
| --- | --- |
| `WF#1 - Deal Risk Intake & Normalize` | Validates the versioned lead contract and emits `idempotencyKey`. |
| `WF#2 - Enrich (Census)` | Adds geographic market context. |
| `WF#3 - Math Engine` | Calculates repair estimate, ARV estimate, maximum offer, and financing baseline. |
| `WF#6 - Scenario & Risk Engine` | Calculates scenario sensitivities, rental metrics, and the risk register. |
| `WF#4 - AI Strategy` | Produces structured marketing/acquisition strategy with a safe fallback. |
| `WF#7 - Human Review Gate` | Builds the human-review checklist and audit payload. |
| `WF#5 - Notion Writer` | Persists the reviewed deal package. |
| `WF0 - Orchestrator` | Public webhook entry point and response builder. |

See [workflows/README.md](workflows/README.md) for the required import order and workflow-ID remapping.

## Lead contract extensions

The new intake workflow accepts all original fields plus:

```json
{
  "strategyMode": "long_term_rental",
  "monthlyRent": 2200,
  "annualTaxes": 3600,
  "annualInsurance": 1800,
  "monthlyHoa": 0,
  "vacancyPct": 0.08,
  "maintenancePct": 0.08,
  "holdingMonths": 6
}
```

`monthlyRent` is required for `long_term_rental` and `brrrr`. Estimates should be explicitly reviewed against property-specific comparables, contractor quotes, and financing terms.

## Local n8n setup

```bash
docker run -d --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n n8nio/n8n
```

Import workflows while inactive. Configure Census, Gemini, and Notion credentials in your own n8n instance; do not commit secrets. Protect the webhook with header authentication before exposing it outside your local network.

## Responsible use

Deal Risk is a decision-support and education workflow. It is not an appraisal, lending system, or automated investment decision maker. Census metrics are area-level context, not property comparables. Every deal requires human review before outreach, pricing, or investment action.

## Attribution

This project is derived from the public `Rayyan-Portfolio/n8n-property-underwriting` demonstration project. Retain attribution and observe the upstream project's terms when publishing a derivative.
