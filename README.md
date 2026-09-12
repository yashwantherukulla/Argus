# Argus

**Explainable Ethereum validator performance monitoring.**

Argus reconstructs Ethereum validator attestations from Beacon-chain data and turns a missed or incorrect vote into a useful operational explanation. Instead of only reporting an effectiveness score, it shows the affected epoch, the protocol condition that failed, the reward component involved, and the likely operational cause.

Developed as a Blockchain Technology course project by **Erukulla Yashwanth** (23BKT0083) and **Mohit Talgotra** (23BKT0075).

## Why Argus

Ethereum validators earn attestation rewards by making correct source, target, and head votes. A lower aggregate score alone cannot explain whether a validator missed its duty, was included too late, voted for an incorrect checkpoint, or voted for a non-canonical block.

Argus provides a per-validator, per-epoch diagnosis. For example:

> Wrong-head vote, included on time. A short reorganisation or block-propagation delay may have caused the validator to observe a different head.

## How it works

```text
Beacon API
  ├─ validator and balance data
  ├─ committee assignments
  ├─ block attestations
  └─ finality and block-header data
          ↓
Attestation analysis service
  ├─ locate the validator in aggregation bits
  ├─ calculate inclusion delay
  ├─ verify source, target, and head conditions
  ├─ calculate Altair reward impact
  └─ classify the miss
          ↓
React dashboard
  ├─ validator summaries
  ├─ epoch-level history
  ├─ trend charts
  └─ error and gap visibility
```

Argus uses beaconcha.in only to resolve public keys and validator indices. The attestation reconstruction and reward calculations use Beacon-node data.

## Attestation checks

| Check | Requirement | Diagnostic meaning |
| --- | --- | --- |
| Source | Correct source checkpoint and inclusion within 5 slots | Late or incorrect source vote |
| Target | Correct epoch target and inclusion within 32 slots | Incorrect target vote |
| Head | Canonical block root and inclusion in the next slot | Wrong-head or late-head vote |

Each epoch is classified as `correct`, `wrong_head`, `wrong_target`, `wrong_source`, `missed_entirely`, `late_source`, or `late_head`.

## Technology

- **Backend:** Node.js, Express, TypeScript
- **Frontend:** React, Vite, Tailwind CSS, Recharts
- **Consensus data:** Ethereum Beacon API
- **Caching:** Redis
- **Observability:** Prometheus and Grafana

## Project structure

```text
Argus/
├── backend/        Express API, Beacon clients, reward analysis, cache
├── client/         React validator-performance dashboard
├── monitoring/     Prometheus and Grafana configuration
└── RESEARCH.md     Methodology, accuracy trade-offs, and references
```

## Run locally

### Backend

```bash
cd backend
cp .env.example .env
# Set BEACON_URL and BEACONCHAIN_API_KEY in .env as needed
pnpm install
pnpm dev
```

The API starts on `http://localhost:3000` by default.

### Frontend

```bash
cd client
npm install
npm run dev
```

If the API is hosted elsewhere, set `VITE_API_BASE_URL` for the frontend environment.

### Optional monitoring stack

```bash
cd monitoring
docker compose up -d
```

## API highlights

- `GET /health` — service and Redis health
- `GET /metrics` — Prometheus metrics
- `GET /api/validators/tracked` — configured validator public keys
- `GET /api/validators/:index/performance` — epoch-level performance for one validator
- `POST /api/validators/batch/performance` — performance for multiple validators

The full request and response contract is documented in [backend/README.md](backend/README.md).

## Validation and research notes

Run the on-chain pipeline check from the backend directory:

```bash
pnpm pipeline
```

The implementation reconstructs participation from attestations included in canonical blocks. This provides an explainable and practical view for recently finalised data, while carrying limits for historical, non-canonical, and full-state-only data. See [RESEARCH.md](RESEARCH.md) for the methodology, assumptions, and literature references.

## Future work

- Validate results across larger validator sets and reference data.
- Improve historical-data support with archival Beacon-node access.
- Add alerts for repeated misses and elevated wrong-head rates.
- Extend analysis to sync-committee participation and inactivity penalties.
