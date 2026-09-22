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
SAST merge condition set in Github rulesets, Require CodeQL Security Checks: Security alerts: High or higher but is currently set to disabled as this is only a mock pipeline.
No artifactory/image repository for built images hence no checks for failures after build/scan. Assumption is that in real world conditions a repository for images will be there and scheduled scans for new CVEs will be done and re-validated before deployment and previously deployed images would be monitored for new CVEs.


The GitHub Actions pipeline applies security checks before the mock deployment:

CodeQL performs SAST to identify potential security issues in the source code.
Trivy performs software composition analysis to identify vulnerable dependencies.
Gitleaks checks the repository for accidentally committed secrets.
Trivy scans the built container image, the pipeline fails when critical vulnerabilities are detected.
Mock OpenShift deployment only runs after security checks pass.

These controls support a shift-left security approach by identifying issues earlier in the development lifecycle. Automated checks provide security validation without requiring every change to undergo manual security review, helping maintain developer velocity while enforcing defined security gates.

PART 4

1. A developer complains that your pipeline's container scan is blocking their release over a "low severity" CVE with no available patch. How do you handle this?

Validate findings by confirming that CVE is applicable to image, determine the impact and exploitability in the specific environment e.g. is the deployed environment open to the internet or is it in an internal air-gapped network. Verify availability of patched version
If no available fix but severity is low check if there are already other mitigating controls, is the vulnerable component actually used by the application? Can it be removed from the image, if it is unused or does it come as a package?
If serverity is low and risk acceptable, allow temporarily on contingency basis with documentation on what the CVE is, the mitigating controls, the actual impact to the application and justification. Pipeline can be adjusted for this specific exception.
Communicate with developer on further possible actions to be taken, monitoring for patches, updates, or possible alternative dependencies.

2. You inherit an OpenShift or AWS account management where every team's ServiceAccount has admin role. Walk us through how you'd remediate this without breaking existing deployments.

Assumption is that removing admin role immediately is not possible without breaking existing deployments.
Work with developers to investigate what permissions are actually being used by the ServiceAccount, utilizing that information define required permissions.
If possible, I would try to recreate a test environment to verify one by one (per workflow) that the defined required permissions I worked to find with the developer are correct.
Remove admin privileges, replace with defined permissions and monitor to ensure everything works as it should. Least-privilege access should be maintained through other controls such as policy-as-code checks moving forward.

3. Briefly describe one real (or realistic) incident where "shifting security left" would have prevented a production issue and how you'd have caught it in the pipeline.

In my experience, most often I would come across issues with dependency vulnerabilities. Generally the cyber security team would conduct scanning on their end to monitor any vulnerabilities in applications being run, and developers would then have to rush to either justify it as a low risk issue or remediate the issue with a patch after the vulnerability was flagged out by the cyber security team. For applications under development such an issue can be flagged out much earlier with SCA tools, monitoring at both the build level and/or at the image repository, allowing developers to know as soon as an issue is brought up without needing to wait for the scheduled updates from cyber security, developers would be able to see that the image would fail at the pipeline build stage before it gets pushed to production.