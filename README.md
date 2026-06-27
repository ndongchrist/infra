# infra

GitOps repo — the **single source of truth for what runs in the cluster**. No
`kubectl apply` by hand in steady state; Argo CD (Phase 10) syncs the cluster to
match this repo. Rollback = `git revert`.

## Layout

```
infra/
├── kafka/                  # Strimzi Kafka cluster, topics, Apicurio registry
│   ├── kafka-cluster.yaml  # single-node KRaft Kafka "ecommerce" (Kafka 4.x)
│   ├── topics.yaml         # KafkaTopic CRs (order.created, payment.*, order.confirmed)
│   └── apicurio.yaml       # Apicurio schema registry (in-memory, dev)
└── (coming) base/ + overlays/dev|prod/   # Kustomize per-service overlays + Argo apps
```

## Current bootstrap (pre-Argo)

Until Argo CD is installed (Phase 10), cluster infra is applied directly:

```bash
kubectl apply -f kafka/kafka-cluster.yaml
kubectl apply -f kafka/topics.yaml
kubectl apply -f kafka/apicurio.yaml
```

Bootstrap = `ecommerce-kafka-bootstrap.kafka:9092` (matches each service chart's
`KAFKA_BOOTSTRAP_SERVERS` default).
