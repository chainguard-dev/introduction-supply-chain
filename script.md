
# software supply chain security

## What is it?

A supply chain is a complex logistical network in which many actors work together over successive steps to bring a finished product to the consumer. In manufacturing, a supply chain takes raw materials as inputs and transforms them into a finished physical product. In software, a supply chain takes hardware and software inputs to develop a software artifact, such as a program or application.

A key element of a modern supply chain is complex interdependence. A finished good such as a television may be fabricated from hundreds or even thousands of "upstream" products such as diodes, transistors, or cables. Each of these components may in turn have their own subcomponents and associated supply chains.

Similarly, modern software artifacts such as applications, libraries, or container images are built out of other software artifacts. For example, when a developer goes to build a modern web application, they are likely to reach for frameworks such as React or libraries such as request. In software, we call these components "dependencies." Each of these dependencies, in turn, can have a supply chain of its own, composed of its own dependencies. These dependencies of dependencies are called "transitive dependencies."

Depending so heavily on "upstream" open source software creates a problem. When dependencies are Incorporated into a software system, vulnerabilities in those dependencies can also affect that system. Incorporating vulnerable dependencies can therefore make your own software vulnerable.

## What is a Vulnerability?

What do we mean when we say a vulnerability?

Vulnerabilities are flaws or weaknesses in artifacts or procedures that open a system to attack. Managing vulnerabilities is important for any kind of infrastructure-grade system, whether a simple web application running on a server or a physical power plant. In software, vulnerabilities can be known, meaning that they have been recognized as affecting a specific system or type of system, or they can be unknown, meaning that the vulnerability has not yet been made public.

Thankfully, when it comes to remediating vulnerabilities in software, we are not starting from scratch. Since 1999, the Common Vulnerabilities and Exposures (CVE) system has cataloged known information security vulnerabilities. This system provides a reference when hardening your infrastructure. The CVE system, in combination with modern devops tooling such as reproducible image builds and automated scanning, has raised the security waterline, making it possible to address known vulnerabilities at scale.

In the modern software security ecosystem, vulnerabilities are tracked in a [National Vulnerability Database (NVD)](https://nvd.nist.gov/) by the [US National Institute of Standards and Technology (NIST)](https://www.nist.gov/). The NVD is a database of Common Vulnerabilities and Exposures (CVE), or known vulnerabilities affecting software. CE are assigned a score, called a CVSS score, that corresponds to the severity of the vulnerability. Based on this score, CVEs can be considered of critical, high, medium, or low severity. 



