## Network Security

* All services run inside Kubernetes as `ClusterIP`, only the Ingress controller is exposed to the public internet.
* Future improvement: Apply `NetworkPolicy` to strictly control which pods/services can communicate with each other.
* Database (MongoDB) is only accessible from backend microservices (auth, discounts, items).

## Secrets Management

* Secrets are stored in **Kubernetes Secrets**, not in `.env` files committed to GitHub.
* Sensitive values include:

  * JWT secret (`SECRET`)
  * MongoDB connection string (`MONGODB_URI`)
  * Cloudinary API keys
* GitHub repository contains **no plaintext secrets**.

## Authentication & Authorization

* Auth microservice issues JWT tokens.
* Discounts and Items microservices validate JWT tokens received in the `Authorization` header.
* Client forwards user credentials to the Auth service only, never directly to other microservices.

## TLS/HTTPS

* TLS is provisioned by **cert-manager** and **Let’s Encrypt**.
* The Ingress enforces HTTPS via annotation `nginx.ingress.kubernetes.io/force-ssl-redirect: "true"`.
* Certificates auto-renew before expiration.
