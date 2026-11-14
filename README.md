# KartShop Helm charts

This repository contains the reusable frontend chart and environment-specific values.

## Layout

- `charts/frontend`: reusable Helm chart (Deployment, ClusterIP Service, Istio Gateway and VirtualService)
- `dev/frontend/values.yaml`: development overrides
- `prod/frontend/values.yaml`: production overrides

## Frontend deployment

Build and push the frontend image to ECR, then deploy with the CI-generated immutable image tag:

```bash
helm upgrade --install frontend ./charts/frontend \
  --namespace dev --create-namespace \
  --values dev/frontend/values.yaml \
  --set image.repository="$ECR_REPOSITORY" \
  --set image.tag="$IMAGE_TAG"
```

For production, use namespace `prod` and `prod/frontend/values.yaml`.

The chart deliberately sets no CPU or memory requests/limits. It configures HTTP readiness and liveness probes against `/_healthz` on port 8080.

## Istio prerequisites

Istio must already be installed in the cluster, with an ingress gateway workload in namespace `istio-system` labelled `istio: ingressgateway`. The chart creates an Istio `Gateway` in `istio-system` and an `VirtualService` in the application namespace. The frontend Service remains `ClusterIP`; external requests reach it through Istio.

The development hostname is `dev.vegz.online`; production uses `vegz.online`. Configure DNS and the KIND/EC2 ingress port path separately. The chart does not install Istio or configure KIND node port mappings.

## Validate

```bash
helm lint ./charts/frontend
helm template frontend ./charts/frontend \
  --namespace dev \
  --values dev/frontend/values.yaml \
  --set image.repository=example/frontend \
  --set image.tag=test
```

The backend service addresses are configurable in `charts/frontend/values.yaml`. The frontend expects those backend microservices to be reachable for full application functionality.
