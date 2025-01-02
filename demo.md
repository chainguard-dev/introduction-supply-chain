In this exercise, we'll scan a container image for vulnerabilities using Grype](https://github.com/anchore/grype), a popular container image scanner released and maintained by [Ancore](https://anchore.com). In this exercise, we'll move through the following steps:

1.  Pull the [Grype container image maintained by Chainguard](https://images.chainguard.dev/directory/image/grype/overview).
2.  Scan the official Python container image for vulnerabilities.
3.  Interpret the results, learning about how CVEs are categorized.

For this exercise, we'll use the [Grype container image maintained by Chainguard](https://images.chainguard.dev/directory/image/grype/overview). However, you may want to consider[ installing Grype](https://github.com/anchore/grype#installation) on your host machine, as it's a useful tool to have in your vulnerability management toolbox. 

To follow this tutiral, you will need a working installation of[ Docker Engine](https://docs.docker.com/engine/install/) or[ Docker Desktop](https://docs.docker.com/desktop/).

## Pulling the Grype Image

Once you have Docker installed, open your terminal and run the following command to pull the Grype container image from Chainguard's image repository:

```sh
docker pull cgr.dev/chainguard/grype
```

Once you have the image, let's run it. This should produce Grype's `--help` output.

```sh
docker run cgr.dev/chainguard/grype
```

We should now be ready to scan a container image using Grype.

## Scanning

To run a scan using Grype, we invoke Grype and pass it the URI (address) of a container image we wish to scan. 

Let's scan the official Python image on Docker Hub:

```sh
docker run -it cgr.dev/chainguard/grype python
```

Before discussing the output results, let's break down the above command in detail:

1. The `docker run -it` command runs the Grype container image interactively with an attached `tty` (terminal interface). If the `-t` is omitted, the dynamic progress indicators will not be displayed and there will be no visible output until the scan is completed. The interactive `-i` flag is not needed, but avoids some initial garbled output on some terminals.
2. The `cgr.dev/chainguard/grype` portion of the command is the address of our Grype container image.
2. The `python` portion of the command is a valid address for the official Python container image hosted on Docker Hub.

Note, you may need to run the command with sudo (sudo docker run -it anchore/grype:latest python) if Docker is not configured to run without root privileges. However, it is strongly recommended to avoid running Docker as the root user.

On running this command, you should see dynamic output from Grype as the official Python image is pulled and scanned. After some time, you should receive a large amount of output. Output from Grype is divided into two parts: an initial overview and an itemized list of specific vulnerabilities. Let's first take a look at the initial overview — you may need to scroll up to find it.

``` 
 ✔ Vulnerability DB                [updated]  
 ✔ Pulled image                    
 ✔ Loaded image                                 python:latest
 ✔ Parsed image                    sha256:7f17499b3a00d0b82c06c
 ✔ Cataloged contents              d763b433394a8b27d7f3642e5d19
   ├── ✔ Packages                        [438 packages]  
   ├── ✔ File digests                    [20,064 files]  
   ├── ✔ File metadata                   [20,064 locations]  
   └── ✔ Executables                     [1,425 executables]  
 ✔ Scanned for vulnerabilities     [623 vulnerability matches] 
   ├── by severity: 12 critical, 91 high, 291 medium, 47 low, 4
   └── by status:   1 fixed, 1277 not-fixed, 655 ignored 
```

Arguably the most important line in this output is the breakdown of vulnerabilities by severity. CVEs are assigned numerical scores according to the[ Common Vulnerability Scoring System (CVSS)](https://nvd.nist.gov/vuln-metrics/cvss). These scores correspond to four categories:

- Critical (9.0-10.0)
- High (7.0-8.9)
- Medium (4.0-6.9)
- Low (0.1-3.9)

The "by severity" line in our Grype output suggests that, as of January 2nd, 2025, the official Python image had 12 critical, 91 high, 291 medium, and 47 low vulnerabilities.

``` language-markup
   ├── by severity: 12 critical, 91 high, 291 medium, 47 low, 495 negligible
```

Another line to look out for is how many of the CVEs have been fixed in a release:

``` language-markup
   └── by status:   1 fixed, 1277 not-fixed, 655 ignored 
```

If you're seeing CVEs with a "fixed" status, that indicates that updating to a new version of the affected package will resolve the issue. In the case of the official Python image, most packages have already been updated to newer versions, so we don't often see any packages that have been marked as fixed. In this case, we have one fixed package in our results that could be mitigated by updating to a newer version.

Grype also itemizes each CVE found during the scan. This constitutes the second half of our output. Let's rerun the scan to look just at the CVE that have critical status:

``` 
docker run -it anchore/grype:latest python | grep Critical
```

This produces the following output:

```
libaom3                       3.6.0-1+deb12u1               (won't fix)  deb     CVE-2023-6879     Critical    
libglib2.0-0                  2.74.6-2+deb12u4              (won't fix)  deb     CVE-2024-52533    Critical    
libglib2.0-bin                2.74.6-2+deb12u4              (won't fix)  deb     CVE-2024-52533    Critical    
libglib2.0-data               2.74.6-2+deb12u4              (won't fix)  deb     CVE-2024-52533    Critical    
libglib2.0-dev                2.74.6-2+deb12u4              (won't fix)  deb     CVE-2024-52533    Critical    
libglib2.0-dev-bin            2.74.6-2+deb12u4              (won't fix)  deb     CVE-2024-52533    Critical    
libopenexr-3-1-30             3.1.5-5                       (won't fix)  deb     CVE-2023-5841     Critical    
libopenexr-dev                3.1.5-5                       (won't fix)  deb     CVE-2023-5841     Critical    
wget                          1.21.3-1+b2                   (won't fix)  deb     CVE-2024-38428    Critical    
zlib1g                        1:1.2.13.dfsg-1               (won't fix)  deb     CVE-2023-45853    Critical    
zlib1g-dev                    1:1.2.13.dfsg-1               (won't fix)  deb     CVE-2023-45853    Critical
```

Note that Grype claimed 12 critical CVEs in the summary, but there are 11 in this output. Grype will remove itemized CVEs if they represent a package installed twice in different locations on the system. This can be confusing, but is the tool's intended behavior.

If you're resolving CVEs manually, this output is your starting point. Generally, you would look up the CVE on[ nist.gov](https://www.nist.gov/), which collects information on affected packages and systems, disclosures, and additional resources for learning about CVEs, such as advisories and potential mitigations. Since the above packages are marked as (`won't fix)`, you will need to either determine that the CVE isn't relevant to your use case, remove the affected package, or manually mitigate the CVE.


## Resources

- [Chainguard Deep Dive 🤿: Where Does Grype Data Come From?](https://dev.to/chainguard/deep-dive-where-does-grype-data-come-from-n9e)
- [Grype on the Anchore blog](https://anchore.com/search/?search=grype) - Blog posts from Anchore related to Grype
- [Why Chainguard uses Grype](https://www.chainguard.dev/unchained/why-chainguard-uses-grype-as-its-first-line-of-defense-for-cves) - Why Chainguard contributes to and recommends Grype for vulnerability scanning in container images
