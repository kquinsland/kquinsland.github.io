<!-- markdownlint-disable-file MD001 -->
# Karl Quinsland - Senior SRE and InfraOps Engineer

mail: contact [AT] [karlquinsland.com](https://karlquinsland.com) | [GitHub - kquinsland](https://github.com/kquinsland/) | [LinkedIn - karlq](https://www.linkedin.com/in/karlq/)

## Summary

Skilled SRE and SysAdmin with a knack for automation in all facets of life.
Firm believer in the power of Infrastructure as Code and [Docs as Code](https://www.writethedocs.org/guide/docs-as-code/).
Passionate about harnessing the potential of generative AI, I actively incorporate it to streamline processes and accelerate outcomes.
As an experienced Python and proficient Rust developer, I continuously evolve my skill set to build and maintain systems that not only meet performance standards but also delight users with their reliability and efficiency.
My ultimate goal is to create solutions that are not just effective but also spark joy.

## Work

### Sr. DevOps Engineer, [Spree3D/SpreeAI](https://spreeai.com/) | 2023-04 - 2024-01

> AWS architect, Incident Management Specialist and versatile DevSecOps professional.
> Focus on automating cloud infrastructure, streamlining Machine Learning deployment on EKS, and enhancing platform observability to ensure system stability at scale.

- Accelerated Terraform apply cycles by ~30x and reduced monthly costs significantly by refactoring a large legacy codebase.
- Designed and implemented a scalable DNS management strategy across AWS regions, facilitating active failover and geographic routing. Led efforts to migrate away from chaotic Route53 sprawl to this manageable and secure setup.
- Established a scalable incident management process, promoting its adoption and standardization.
- Containerized multiple CUDA powered inference workloads for deployment on EKS using home-grown tooling and KubeFlow.
- Rolled out observability stack (Loki, Grafana, Tempo, and Mimir) across dozens of EC2/RDS workloads, EKS clusters and lambda functions.
- Leveraging LGTM stack, created dashboards to guide application and platform stability initiatives.
- Mentored the ML team on production-grade Python coding patterns, development tools, and workflows.
- Simplified the onboarding process through comprehensive automation and documentation.

### Sr. Ops Engineer/SRE, [Sprig](https://sprig.com/) | 2022-04 - 2023-02

> In-house AWS specialist tasked with taming legacy Terraform, modernizing kubernetes tooling, workflows and configuration. Provided support to ML team and general 'as-needed' tool development / process automation.

- Reduced monthly AWS costs by thousands through Terraform modernization and infrastructure consolidation.
- Architected and deployed highly scalable CI/CD pipe for both IaC and Lambda functions; fastest "merged" to "deployed and taking traffic" was sub 20s.
- Wrote extensive documentation for - and provided training on - new IaC tooling. Automated document generation for existing IaC leading to large reduction in dev time to provision traditional and EKS based infrastructure.
- Implemented a standardized ADR (architecture design review) process, enhancing architectural discussions and documentation through regular coaching.
- Spearheaded efforts to eliminate nuisance pages and standardized incident management, improving recovery speed and cost.
- Managed the bug bounty program, triaging and prioritizing remediation efforts across teams.
- Performed security focused code and architecture review for new product features.

### Sr. Ops Engineer/SRE, [Kiddom](https://www.kiddom.co/) | 2020-10 - 2022-04

> In house Terraform specialist tasked with refactoring legacy Terraform and
automating Cassandra cluster management.
> Evangelized Infrastructure as Code practices and championed observability initiatives.

- Tool-smith: Wrote more than a few scripts/tools/lambda functions to automate everything from Cassandra cluster assembly to management of DNS entries and TLS Certificate provisioning to documentation generation.
- OSS contributor: new features and bug fixes merged into various Cassandra management tools and Terraform providers.
- Led efforts to increase observability into applications and infrastructure and build out a mature incident reporting and management apparatus.
- Overhauled IAM and introduced best practices for cross-account resource utilization... and managed it with Terraform.
- Brought IaC practices to as many things as possible including external services; onboarding new Eng. hires was done with a few PRs and tools like Atlantis did the heavy lifting.
- Developed extensive corpus of documentation around IaC driven processes as well as legacy infrastructure and it's replacement - both planned and implemented. First dedicated Ops/Infra/SRE hire. Taming unwieldy infrastructure, porting legacy Terraform, developing/modernizing tooling and practices and leading efforts to instrument all layers of the stack.
- Initiated the transition from legacy bare-metal /auto-scaled AMIs based architecture to an EKS based platform.

### Sr. Production Engineer/SRE, [Touchpoint Restaurant Innovations](https://touchpoint.io) | 2018-10 - 2020-04

> Built tooling to formalize and automate the deployment of networking hardware at customer sites. Oversaw the transition to a containerized infrastructure and led the migration to Elastic Search 7.

- Tool-smith: Dozens of in-house tools some of which radically changed (read: streamlined and automated) how hardware was shipped to customers. Other tools developed as needed to rectify problems at hundreds of in-field deployments.
- Hashicorp evangelist. Terraform, Packer, Consul, Vault used/deployed to make for a largely immutable, smarter, more secure operating environment.
- Vocal advocate for the containerization of workloads whenever possible. Strong proponent of immutable infrastructure and declarative infrastructure as code.
- Tamer of Elastic Search (Open Distro for Elastic Search on Docker, specifically). Worked w/ Application Engineering side of the house on migration to ES7.
- Supervisor of in-field hardware; everything from instrumenting for prometheus and building Grafana dashboards to vetting OEM samples to reverse engineering troublesome firmware and proprietary APIs. equal parts troubleshooter, automation tooling engineer, AWS architect. Owned efforts to modernize, internalize, containerize various bits of core infrastructure.

### Sr. SRE, DevOps evangelist, [Eventbrite (via Ticketfly)](https://www.eventbrite.com/)

##### 2017-09-01 - 2018-10-01

- Worked to revise how Eventbrite and Ticketfly respond to critical incidences.
- Wrote tooling to better link Vault to Eventbrite's central authentication flow.
- Modernization of existing Puppet manifests.
- Coached colleagues on networking in AWS and Terraform best practices.
- Created networking components to securely share data between EB and TF data lakes; extensive work w/ EB DataScience team to fine-tune performance. Support Ticketfly's systems and help Eventbrite build new tools and processes to give developers more insight into their systems and the organization more flexibility

### Sr. SRE, DevOps Practitioner, Tools Engineer, Cloud former, [Ticketfly](https://www.ticketfly.com/)

##### 2015-01-01 - 2017-09-01

- Was 24x7 Tier 4 OnCall
- Rebuild a modern / skilled DevOps team.
- Significant tech-debt eradication, ancient stack archaeology, platform spelunking.
- Authored countless Chef cookbooks, ruby gems.
- SCALR server administrator.
- Mongo and MySQL dbe/a.
- Consult as an SRE and Architect for new apps and services.
- Successfully ran bare metal / data center exit & equipment disposition & data destruction efforts.
- Designed networking architecture for entirety of cloud deployment, negotiated and set up private fiber circuits, VPN tunnels and automated other security controls. Hired as in the in-house AWS Expert. Led the migration from dataccentre to AWS and drove the necessary infrastructure and application architecture changes needed to pull it off.

### Associate Software Engineer, [Acxiom](https://www.ticketfly.com/)

##### 2014-02-01 - 2015-01-01

- Developed chef training / coached internal teams on migration to Chef.
- Built tooling to automate administration of Aditive software and better integrate it within Acxiom's AWS environments.
- Worked with colleagues in Asia to support and roll out Aditive software for Asian markets.
- SRE, Ops, Dev work for a variety of internal and external applications, some of which serve millions of requests every day. Principal support contact for all Aditive software. Integral Chef 'consultant'.

### DevOps Engineer, [Aditive](https://www.aditive.com/)

##### 2013-07-01 - 2014-02-01

- Single handedly engineered and implemented growth/scale strategy for high demand web services.
- Responsible for most infrastructure maintenance, including upgrades to production appliances and live migrations to AWS/VPC from AWS/EC2 Classic.
- Designed and built a variety of deployment tools to aide under staffed development team with single-click / continuous deployment for testing and production environments. Equal parts Mongo and Vertica 5 DBA. Led development of software to automate build/deploy of Aditive software in A/B style. In-house Chef dev.

## Education

### UC, Merced - BS, Computer Science & Engineering

#### 2009-08-01 - 2013-05-01

- GPA: Deans List, multiple semesters

##### Courses

- Computer Security
- Networking
- Computer Architecture
- Databases
- Algorithms

## Awards

- Amateur Radio Operator from FCC on 2020-01-01
  - General Class
- Intermediate Vault Training from HashiCorp on 2016-09-01
  - Everything needed to deploy, run, integrate Vault

## Skills

### Programming Languages

- Python3: advanced
- golang: passable
- rust: new, liking a lot!
- php: rusty
- ruby: rusty
- C/++: hacky

### Cloud Providers

- AWS: advanced / 10+ years experience.
- DigitalOcean: intermediate. 3+ years experience.
- Azure/Google: passable

### Containerization

- Docker: advanced
- kubernetes/k8s: advanced / 5+ years experience.

### K8s/IaC/Ops Tools

- Terraform/[OpenTofu](https://opentofu.org/): advanced / 9+ years experience.
- Argo/Flux/LGTM: advanced
- Packer: advanced
- Consul: advanced
- Vault: advanced

### General SRE/Ops/Admin

- Networking: duties ranging from setting up dedicated private circuits to troubleshooting latency to designed and automated VPC networking and associated network security controls
- troubleshooting: advanced+
- performance tuning: intermediate
- testing: intermediate

## Languages

- 🇺🇸 English: Native  English
- 🇪🇸 Spanish: No asombroso, suficiente

## Interests

- Fitness (avid gym rat, automated nutrition optimization, practitioner of various mindfulness techniques)
- Home/Personal Automation (built/hacked several IoT devices; see [https://karlquinsland.com/tags/esphome)](https://karlquinsland.com/tags/esphome/)
- Networks & wireless systems (Licensed HAM (we're allowed to build / test our own wireless protocols), Passionate student of all techniques/protocols for linking systems together)

## References

Provided on request
