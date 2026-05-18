# Kubernetes Launch

This bundle runs the Miles ROCm Qwen3-30B-A3B experiment on one 8-GPU AMD
Kubernetes node. It uses a retained hostPath PVC at
`/scratch/vunguyen13/miles-rocm-article-qwen3-30b-a3b` on node `vu` for models,
datasets, converted checkpoints, run logs, and training checkpoints.

## Files

- `00-volume.yaml`: namespace, retained hostPath PV, and PVC.
- `01-configmaps.yaml`: shared environment and launch scripts.
- `02-debug-pod.yaml`: optional shell pod with the PVC mounted at `/mnt/miles`.
- `03-training-job.yaml`: training Job with an `asset-prep` init container and `trainer` container.

## Launch

```bash
cd sdks/mlops/playground/experiments/2026-05-14-miles-rocm-article-qwen3-30b-a3b
kubectl apply -f k8s/00-volume.yaml
kubectl apply -f k8s/01-configmaps.yaml
kubectl apply -f k8s/03-training-job.yaml
```

The `asset-prep` init container downloads `Qwen/Qwen3-30B-A3B`,
`zhuzilin/dapo-math-17k`, and `zhuzilin/aime-2024`, converts the HF checkpoint
to Megatron `torch_dist`, then writes `/mnt/miles/state/qwen3-30b-a3b.assets-ready`.
The trainer starts only after that prep container exits successfully, then still
checks the sentinel before starting Ray and Miles.

Both `asset-prep` and `trainer` request `amd.com/gpu: "8"`, tolerate the
`gpu=true` node taint, and mount `/dev/kfd` plus `/dev/dri` because checkpoint
conversion also needs HIP GPU access. `asset-prep` is an init container so
conversion owns the GPUs before training starts. The ROCm visibility variables
allow devices `0,1,2,3,4,5,6,7`, and Ray starts with `RAY_NUM_GPUS=8`.

Use the debug pod only when you need to inspect the shared volume:

```bash
kubectl apply -f k8s/02-debug-pod.yaml
kubectl exec -n vunguyen13 -it miles-rocm-article-qwen3-30b-a3b-shell -- bash
```

## Runtime Overrides

The default `RUN_MODE` is `canary`. Edit `RUN_MODE` in `01-configmaps.yaml` to
`minbar` or `article`, then recreate the ConfigMap and Job:

```bash
kubectl delete job -n vunguyen13 miles-rocm-article-qwen3-30b-a3b-train
kubectl apply -f k8s/01-configmaps.yaml
kubectl apply -f k8s/03-training-job.yaml
```

For gated models or Weights & Biases, create an optional secret:

```bash
kubectl create secret generic miles-runtime-secrets -n vunguyen13 \
  --from-literal=HF_TOKEN=<token> \
  --from-literal=WANDB_API_KEY=<key>
```

## Logs

```bash
kubectl logs -n vunguyen13 job/miles-rocm-article-qwen3-30b-a3b-train -c asset-prep -f
kubectl logs -n vunguyen13 job/miles-rocm-article-qwen3-30b-a3b-train -c trainer -f
```
