## Data Handling & Storage

* **MongoDB** stores user data. When deployed on cloud infrastructure, encryption at rest is enabled by default (cloud provider responsibility).
* JWT tokens are stateless and only stored client-side (browser/local storage).
* Passwords are stored in MongoDB only after hashing (e.g., bcrypt or similar).

## GDPR / User Privacy

* Personally Identifiable Information (PII) is limited to user account details (email, name, etc.).
* Users can request account deletion, which will remove their data from MongoDB.

## Security Practices

* All traffic is served via HTTPS.
* Secrets are never exposed in logs or public repositories.
* Role-based access is applied at microservice level:

  * Auth service validates and signs JWT.
  * Other services validate the token before executing restricted actions.

## Future Improvements

* Implement audit logging for sensitive operations.
* Consider deploying MongoDB with field-level encryption if compliance frameworks (HIPAA, PCI) are required.
* Regular penetration testing and vulnerability scans.
