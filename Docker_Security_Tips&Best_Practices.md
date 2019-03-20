## Docker Security Tips & Best Practices

The strengths of containerization come hand in hand with critical challenges for developers and operational teams. Given the more complex architecture and multiple services that need to be tackled when Docker is being deployed, the areas of concern can be better understood if they are broadly classified as follows:

* Monitoring and Visibility: Docker developers use a variety of command line tools to manage their applications and hand over containers to operations with minimal monitoring integrated into them.
* Configuration and Control: Challenges around the support system with Docker include resource pooling, access policies, load balancing, and application lifecycle management.
* Integration and Security: Integration problems can exist, centering on the need to create a cloud dev/test environment and an on-site dev/test environment.

On the security front, developers are faced with different types of security attacks such as:

* Kernel exploits: Since the host’s kernel is shared in the container, a compromised container can attack the entire host.
* Container breakouts: Caused when the user is able to escape the container namespace and interact with other processes on the host.
* Denial-of-service attacks: Occur when some containers take up enough resources to hamper the functioning of other applications.
* Poisoned images: Caused when an untrusted image is being run and a hacker is able to access application data and, potentially, the host itself.

Recently, we talked to ZDNet about how Docker containers are now being exploited to covertly mine for cryptocurrency, marking a shift from ransomware to cryptocurrency malware. As with all things security, Docker security is a moving target — so it’s helpful to have access to up-to-date information, including experience-based best practices, for securing your containerized environments.

### Docker Security Tips
Below, we’ll discuss a few important tips you should keep in mind when implementing security with Docker:

#### 1. Use a Third-Party Security Tool
Docker allows you to use containers from untrusted public repositories, which increases the need to scrutinize whether the container was created securely and whether it is free of any corrupt or malicious files. For this, use a multi-purpose security tool that gives extensive dev-to-production security controls.

#### 2. Manage Vulnerability
It is best to have a sound vulnerability management program that has multiple checks throughout the container lifecycle. Vulnerability management should incorporate quality gates to detect access issues and weaknesses for a potential exploit from dev-to-production environments.

#### 3. Monitor and Audit Container Activity
It is vital to monitor the container ecosystem and detect suspicious activity. Container monitoring activities provide real-time reports that can help you react promptly to a security breach.

#### 4. Enable Docker Content Trust
Docker Content Trust is a new feature incorporated into Docker 1.8. It is disabled by default, but once enabled, allows you to verify the integrity, authenticity, and publication date of all Docker images from the Docker Hub Registry.

#### 5. Use Docker Bench for Security
You should consider Docker Bench for Security as your must-use script. Once the script is run, you will notice a lot of information regarding configuration best practices for deploying Docker containers that can be used to further secure your Docker server and containers.

### Docker Security Best Practices
Here are some of the best practices you need to follow to ensure security with Docker:

#### 1. Lifecycle Management
Docker security primarily relies on how you handle the container lifecycle starting with creating, updating, and finally deleting containers. We strongly recommend that when updating a container, you test the entire stack from a security perspective instead of just the updated layer.

#### 2. Information Management
Sensitive information such as secrets (e.g., SSH keys, passwords, tokens, TLS certificates) need to be encrypted and stored in a Secrets Manager (e.g., Docker Swarm, HashiCorp Vault) and not at the host level. A secret should be accessible by a service only if it has been granted access explicitly and only when the service is running.

#### 3. Access Control
Having a well-established access management solution for Docker is a must so that containers can operate with minimal privileges and access, which helps reduce risk. Large organizations can incorporate role-based access control (RBAC) and use directory solutions such as Active Directory to manage permissions for all personnel of the organization.

#### 4. Docker Image Authentication
Checking the authenticity of all images before downloading them from untrusted sources is essential. In order to avoid security vulnerabilities, always use base images that are reviewed and scanned by Docker’s Security Scanning Services, or use a base image that is digitally signed by Docker Content Trust.

#### 5. Resource Utilization
To reduce performance impacts and denial-of-service attacks, it is a good practice to implement limits on the system resources that the containers can consume. If, for example, a web server is compromised, it helps to limit the impact to the other processes that are running on a host.

### Wrapping Up . . .
Docker won’t fix all your problems in the cloud, but it does offer myriad benefits for the right applications. If you’re moving to containers, Docker may be just what you’re looking for — but don’t lose sight of the importance of securing your containerized environments. Threat Stack offers Docker integration to help you make smarter security decisions at the same time that you’re streamlining your data consumption. (In fact, Threat Stack can even be deployed as a container.)
