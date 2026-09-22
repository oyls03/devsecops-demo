PART 2
This is a theoretical deployment for OpenShift and not tested in an OpenShift environment.

The application and OpenShift configuration include the following security controls:

Docker image uses the official node.js alpine image and installs only production dependencies.
Application runs as non-root "node" user.
Container privilege escalation is disabled.
All linux capabilities are dropped.
CPU/memory requests are limited.
Default runtime seccomp profile is enabled.
A dedicated openShift ServiceAccount is used.
RBAC is namespace-scoped and provides only limited read-only access to pods.
NetworkPolicy restricts ingress traffic to the application.
Application is exposed through an openshift route with HTTPS and HTTP-to-HTTPS redirection.
No secrets or credentials are included in the source code or container image.
The container image reference is currently a placeholder for this demonstration.

PART 3

A few assumptions and intentional designs
For the pipeline the Docker image is built twice, once for the build job and once for the container-scan job, this is to keep the pipeline as simple as possible.
SAST merge condition set in Github rulesets, Require CodeQL Security Checks: Security alerts: High or higher
No artifactory/image repository for built images hence no checks for failures after build/scan. Assumption is that in real world conditions a repository for images will be there and scheduled scans for new CVEs will be done and re-validated before deployment and previously deployed images would be monitored for new CVEs.


The GitHub Actions pipeline applies security checks before the mock deployment:

CodeQL performs SAST to identify potential security issues in the source code.
Trivy performs software composition analysis to identify vulnerable dependencies.
Gitleaks checks the repository for accidentally committed secrets.
Trivy scans the built container image, the pipeline fails when critical vulnerabilities are detected.
Mock OpenShift deployment only runs after security checks pass.

These controls support a shift-left security approach by identifying issues earlier in the development lifecycle. Automated checks provide security validation without requiring every change to undergo manual security review, helping maintain developer velocity while enforcing defined security gates.