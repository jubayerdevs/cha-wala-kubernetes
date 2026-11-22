# Cha-Wala on Kubernetes ☕

Beginner-friendly Kubernetes assignment project using Minikube, Docker, kubectl, ConfigMaps, Secrets, Deployments, Services, and rollout troubleshooting.

## Project structure

```text
cha-wala-kubernetes/
├── namespace.yaml
├── cha-config.yaml
├── cha-secret.yaml
├── cha-deployment.yaml
├── cha-service.yaml
├── cha-broken.yaml       # corrected debug example
├── menu.html             # standalone reference menu
├── README.md
├── .gitignore
└── screenshots/           # add your assignment screenshots here
```

## 1. Start Minikube

```bash
minikube start --driver=docker --cpus=2 --memory=4096
kubectl cluster-info
kubectl get nodes -o wide
```

## 2. Create namespace and resources

```bash
kubectl apply -f namespace.yaml
kubectl apply -f cha-config.yaml
kubectl apply -f cha-secret.yaml
kubectl apply -f cha-deployment.yaml
kubectl apply -f cha-service.yaml
```

Check status:

```bash
kubectl get all -n cha-express
kubectl get configmap,secret -n cha-express
kubectl get endpoints cha-service -n cha-express
```

## 3. View the menu

```bash
minikube service cha-service -n cha-express
```

This opens the service in a browser. If needed, use the URL printed by the command.

## 4. Verify environment variables

```bash
kubectl get pods -n cha-express
kubectl exec POD_NAME -n cha-express -- sh -c 'printenv SHOP_NAME; printenv CITY; printenv THEME_COLOR; printenv OPENING_HOURS; printenv ADMIN_PASSWORD'
```

Replace `POD_NAME` with a Pod name from `kubectl get pods`.

## 5. Secret note

The values in `cha-secret.yaml` are dummy values for lab use only. Kubernetes Secret values are base64-encoded in the standard YAML representation; base64 is not encryption. Encryption-at-rest depends on cluster configuration. Do not commit real passwords or API keys.

## 6. Rolling update and rollback

```bash
kubectl annotate deployment cha-deployment kubernetes.io/change-cause="Upgrade Nginx from 1.25-alpine to 1.27-alpine" -n cha-express --overwrite
kubectl set image deployment/cha-deployment cha-web=nginx:1.27-alpine -n cha-express
kubectl rollout status deployment/cha-deployment -n cha-express
kubectl rollout history deployment/cha-deployment -n cha-express
```

For a lab rollback test, set an invalid image tag:

```bash
kubectl set image deployment/cha-deployment cha-web=nginx:1.27-invalid -n cha-express
kubectl get pods -n cha-express
kubectl describe pod POD_NAME -n cha-express
kubectl rollout undo deployment/cha-deployment -n cha-express
kubectl rollout status deployment/cha-deployment -n cha-express
```

## 7. Debugging example

`cha-broken.yaml` is the corrected version of the debug workload. The four original issues to diagnose are:

1. Image tag typo (`nginx:1.27-alpinee`) causing image pull failure.
2. ConfigMap name typo (`cha-confg`) causing a missing ConfigMap reference.
3. Service selector mismatch (`app: cha-brokn`) preventing it from selecting the Pods.
4. Service target port 8080 instead of Nginx port 80.

Apply the corrected example:

```bash
kubectl apply -f cha-broken.yaml
kubectl get pods,service,endpoints -n cha-express
```

## 8. ConfigMap bonus

Update `THEME_COLOR` and the CSS color in `index.html` within `cha-config.yaml`, then apply:

```bash
kubectl apply -f cha-config.yaml
```

A mounted ConfigMap file (without `subPath`) is updated eventually by Kubernetes. Environment variables sourced from a ConfigMap do not change inside an already-running container. Restart the Deployment to refresh its environment:

```bash
kubectl rollout restart deployment/cha-deployment -n cha-express
kubectl rollout status deployment/cha-deployment -n cha-express
```

## 9. Screenshots

Place your own screenshots in the `screenshots/` folder. Suggested names:

- `01-node-info.png`
- `02-control-plane.png`
- `03-configmap.png`
- `04-secret.png`
- `05-deployment.png`
- `06-menu-browser.png`
- `07-environment-variables.png`
- `08-copilot.png`
- `09-rolling-update.png`
- `10-rollback.png`
- `11-debug-fixed.png`
- `12-configmap-bonus.png`

## 10. GitHub submission

```bash
git init
git add .
git status
git commit -m "Complete Cha-Wala Kubernetes assignment"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/cha-wala-kubernetes.git
git push -u origin main
```

Create the repository on GitHub first. Use your own username in the remote URL. Never push real credentials.
