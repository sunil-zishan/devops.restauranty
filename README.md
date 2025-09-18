# 🍴 Restauranty – DevOps Setup

This repository contains the infrastructure and deployment setup for the Restauranty application, a microservices-based project with authentication, discounts, items, and a React client frontend, backed by MongoDB and served through NGINX Ingress.

---

## Project Structure

```
.
├── backend/
│   ├── auth/          # Auth service (Node.js + Express + MongoDB)
│   ├── discounts/     # Discounts service
│   ├── items/         # Items service
│
├── client/            # React frontend served with NGINX
│
├── k8s/
│   ├── base/          # Namespace + Ingress manifests
│   ├── services/      # Service manifests
│   │   ├── backend/   # Auth, Discounts, Items deployments + services
│   │   ├── frontend/  # Client deployment + service
│   │   └── db/        # MongoDB deployment + service
│   ├── secrets/       # JWT & Mongo secrets
│
└── .github/workflows/ci-cd.yml  # GitHub Actions pipeline
```

---

## Docker

Each service has its own `Dockerfile`.

### Example: Backend (Auth)

```
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
EXPOSE 3001
CMD ["npm", "start"]
```

### Example: Frontend (Client)

```
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
ARG REACT_APP_SERVER_URL
ENV REACT_APP_SERVER_URL=$REACT_APP_SERVER_URL
COPY . .
RUN npm run build

FROM nginx:alpine
RUN rm -rf /usr/share/nginx/html/*
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

* Multi-stage build ensures optimized frontend images.
* `REACT_APP_SERVER_URL` is injected at build time.

---

## Kubernetes

### Namespace

```
apiVersion: v1
kind: Namespace
metadata:
  name: sunil-restauranty-dev
```

### Services & Deployments

* Auth (`auth-deployment`, `auth-service`) → port 3001
* Discounts (`discounts-deployment`, `discounts-service`) → port 3002
* Items (`items-deployment`, `items-service`) → port 3003
* Client (`client-deployment`, `client-service`) → port 80
* MongoDB (`mongo-deployment`, `mongo-service`) → port 27017

### Ingress

Routes traffic from `http://sunilzishan.com` to the appropriate service:

```
rules:
  - host: sunilzishan.com
    http:
      paths:
        - path: /api/auth
          pathType: Prefix
          backend:
            service:
              name: auth-service
              port:
                number: 3001
        - path: /api/discounts
          pathType: Prefix
          backend:
            service:
              name: discounts-service
              port:
                number: 3002
        - path: /api/items
          pathType: Prefix
          backend:
            service:
              name: items-service
              port:
                number: 3003
        - path: /
          pathType: Prefix
          backend:
            service:
              name: client-service
              port:
                number: 80
```

---

## CI/CD with GitHub Actions

Workflow file: `.github/workflows/ci-cd.yml`

### Highlights

* Triggered on push to `main`.
* Uses Docker Buildx for multi-arch builds.
* Images tagged with the current Git SHA:

  * `sunilzishan/auth:${GITHUB_SHA}`
  * `sunilzishan/discounts:${GITHUB_SHA}`
  * `sunilzishan/items:${GITHUB_SHA}`
  * `sunilzishan/client:${GITHUB_SHA}`
* Deployments are updated via `kubectl set image`.
* Manifests applied in order:

  1. Namespace
  2. Services & Deployments
  3. MongoDB
  4. Ingress

---

## Secrets Required

Stored in GitHub Repository Secrets:

* `DOCKERHUB_USER` → Docker Hub username
* `DOCKERHUB_TOKEN` → Docker Hub access token
* `KUBECONFIG_DATA` → Base64-encoded kubeconfig for the AKS cluster

---

## Deploy Manually (without CI/CD)

Build & push an image:

```
docker buildx build --platform linux/amd64 -t sunilzishan/auth:dev ./backend/auth --push
```

Apply manifests:

```
kubectl apply -f k8s/base/namespace.yaml
kubectl apply -f k8s/services/backend/auth/auth-manifest.yaml -n sunil-restauranty-dev
kubectl apply -f k8s/base/ingress.yaml -n sunil-restauranty-dev
```

Update image in deployment:

```
kubectl set image deployment/auth-deployment auth=sunilzishan/auth:dev -n sunil-restauranty-dev
```

---

## Monitoring & Debugging

* Check pods:

```
kubectl get pods -n sunil-restauranty-dev
```

* Logs:

```
kubectl logs deploy/auth-deployment -n sunil-restauranty-dev
```

* Test ingress:

```
curl -vk http://sunilzishan.com/api/auth/login
```

---

## Summary

This repo integrates:

* Dockerized microservices
* Kubernetes deployments
* NGINX Ingress
* GitHub Actions CI/CD
* MongoDB backing service

All deployments are tagged with immutable Git SHAs, ensuring reproducibility and safe rollbacks.
