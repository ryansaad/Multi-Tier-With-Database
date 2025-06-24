# Automated Deployment of a Multi-Tier Application to AWS EKS with Terraform

## Project Overview

This project establishes a complete CI/CD pipeline and Infrastructure as Code (**IaC**) workflow to deploy a multi-tier Java application to an AWS EKS Kubernetes cluster. It also demonstrates how to secure the application with HTTPS by configuring a custom domain and SSL certificates.

## Key Features

* Provisioning the AWS EKS cluster and all necessary infrastructure using **Terraform (IaC)**.
* Setting up and configuring key DevOps tools like **Jenkins**, **Nexus**, and **SonarQube**.
* Automating the build, test, and security analysis of the multi-tier application.
* Orchestrating the application's deployment to the **Kubernetes cluster**.
* Configuring a custom domain to map to the application's load balancer URL.
* Securing the application's URL with **SSL certificates** for **HTTPS**.


## Technologies Used

* **AWS (Amazon Web Services):** The cloud provider used to host all infrastructure and services, including EC2 and EKS.
* **Terraform:** Used for **Infrastructure as Code (IaC)** to provision the AWS EKS Kubernetes cluster and EC2 instances in a repeatable and automated manner.
* **Jenkins:** The central CI/CD automation server that orchestrates the entire pipeline, from code build to Kubernetes deployment.
* **Kubernetes (EKS):** The container orchestration platform used to deploy, manage, and scale the multi-tier application and its database.
* **Maven:** The build automation tool used for compiling the Java application, running tests, and creating the deployable artifact.
* **SonarQube:** Integrated into the pipeline for **Static Application Security Testing (SAST)** to analyze code quality and vulnerabilities.
* **Nexus Repository Manager:** Used for **artifact management**, serving as a repository for the built application's JAR file.
* **Trivy:** A vulnerability scanner used to scan Docker images for security vulnerabilities before deployment.
* **GoDaddy DNS:** The DNS management service used to configure a custom domain and map it to the Kubernetes load balancer URL.
* **Cloudflare:** A service used to manage DNS records and provision **SSL certificates** to secure the custom domain with **HTTPS**.
* **Docker:** Used to containerize the multi-tier application and its MySQL database.
* **Groovy:** The language used to write the Jenkins Pipeline (Jenkinsfile).
* **MySQL:** The relational database used by the application, deployed within the Kubernetes cluster.



## Architecture / Workflow

The pipeline is designed to automate the software delivery lifecycle with integrated security.

1.  **Infrastructure Provisioning (IaC):** **Terraform** is used to create a central VM and then provision a complete **AWS EKS Kubernetes cluster**, including worker nodes, in a repeatable and automated manner.
2.  **Tool Setup:** Separate EC2 instances are provisioned and configured to host **Jenkins**, **SonarQube**, and **Nexus Repository Manager**.
3.  **Application Build & Analysis (CI):** A developer commits code, triggering the **Jenkins** pipeline. The pipeline fetches the multi-tier application's source code (**Java/Maven**) from GitHub, uses **Maven** to build the application, and then **SonarQube** performs static code analysis. The application artifact is published to **Nexus**.
4.  **Container Orchestration & Deployment (CD):** The application is deployed to the **AWS EKS cluster** using **Kubernetes** manifests. **RBAC (Role-Based Access Control)** is configured to ensure that only a specific service account has permission to perform deployments.
5.  **Post-Deployment Access & Security:** The application is exposed via a **Kubernetes Load Balancer**, providing a URL for access. A custom domain (**GoDaddy DNS**) is mapped to the Load Balancer URL using a **CNAME** record, and **Cloudflare** is used to provision **SSL certificates** to secure the custom domain with **HTTPS**.














## Challenges and Solutions

* **Understanding DNS Records for Domain Mapping:**
    * **Challenge:** I found it difficult to understand the difference between an **A record** and a **CNAME record** when mapping my custom domain in GoDaddy. I wasn't sure which one to use to point my domain to the load balancer URL provided by the application's deployment.
    * **Solution:** Through troubleshooting and research, I learned that a **CNAME record** is used to map a domain to another hostname or URL (like a load balancer URL), while an **A record** maps to a static IP address. This clarified the correct DNS configuration and enabled the domain to successfully point to my application.
* **SSL Certificate Activation and DNS Propagation Delay:**
    * **Challenge:** After adding my domain to **Cloudflare** for a free SSL certificate, the application was still accessible over HTTPS and showed as "Not Secure." The certificate wasn't activating as expected.
    * **Solution:** I discovered that the final step to activate the SSL certificate required changing my domain's nameservers in GoDaddy to the ones provided by Cloudflare. This crucial step hands over DNS control and initiates the certificate provisioning. I also learned about **DNS propagation**, realizing that changes can take several minutes to reflect globally. This taught me patience and a key networking concept for live deployments.
* **Maven Artifact Publishing to Nexus:**
    * **Challenge:** The Maven build could not publish the application artifact to the **Nexus** repository due to a missing credentials configuration. Maven was unable to authenticate with Nexus.
    * **Solution:** I learned that Maven needs explicit server credentials defined outside the `pom.xml` for security. I configured a `settings.xml` file within Jenkins' `Managed files` and linked it to the pipeline using `withMaven`, which provided the necessary Nexus username and password for successful authentication.

---

## Lessons Learned

* **Infrastructure as Code (IaC):** Gained hands-on expertise in using **Terraform** to provision a complete cloud infrastructure stack, including a **Kubernetes** cluster and virtual machines.
* **CI/CD Orchestration:** Solidified skills in using **Jenkins** to automate the entire software delivery lifecycle, from code build to deployment.
* **DevSecOps Integration:** Understood the critical importance of integrating security tools like **SonarQube** into the pipeline to identify vulnerabilities early.
* **Networking and DNS:** Developed a crucial understanding of DNS concepts, including the difference between **A** and **CNAME** records, and the process of configuring custom domains and managing name servers in services like **GoDaddy** and **Cloudflare**.
* **Problem-Solving and Troubleshooting:** Learned to debug complex pipeline failures by analyzing build logs in **Jenkins** and developed skills in diagnosing networking issues, such as **DNS propagation delays**, a common challenge in live environments.
* **New Insights Gained:**
    * The significant impact of initial setup times for security tools (like database downloads) on the first build, and how subsequent runs benefit from caching.
    * The importance of precise build commands (e.g., `mvn clean package` vs. `mvn clean compile`) for successful artifact generation.

---

## Future Enhancements

* **Automated Webhook Triggers:** Implement **GitHub webhooks** to automatically trigger the **Jenkins** pipeline on every code commit, moving from a manual trigger to a fully automated **CI** workflow.
* **Advanced Security Gates:** Configure the pipeline to automatically fail a build if **SonarQube** or other security tools detect critical vulnerabilities, enforcing a true "fail-fast" **DevSecOps** approach.
* **Dynamic Deployment:** Use a more dynamic deployment strategy (e.g., **Helm charts**) to manage **Kubernetes** deployments more efficiently and allow for easy rollbacks.
* **Monitoring and Logging:** Integrate monitoring tools like **Prometheus** and **Grafana** to collect metrics from the **EKS** cluster and application, and implement a centralized logging solution (e.g., **ELK stack**) for better observability.
* **Cloud-Native Storage:** Use persistent volumes in **Kubernetes** for the database, ensuring data is not lost on pod restarts.
* **High Availability:** Implement a multi-zone **EKS cluster** configuration for high availability and disaster recovery.
