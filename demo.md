# Demo: Migrating a Python Application to Chainguard Images

In this demo, we'll migrate a Dockerfile for a Python command line application to Chainguard Images. Before and after the migration, we'll scan our build for known vulnerabilities (CVEs) using [Grype](https://github.com/anchore/grype), a popular container image scanner.

We'll be moving through the following steps:

1. Install required software (Docker and Grype).
2. Create a sample CLI application using a Dockerfile building on the [official Python base image](https://hub.docker.com/_/python/).
3. Scan the image with Grype to find out how many CVEs are present.
4. Migrate the Dockerfile to use the [Python Chainguard Image](https://images.chainguard.dev/directory/image/python/overview) as the base image.
5. Run the scanner again to see if the CVE situation has improved. 😇

After the demo, we'll share some resources for those who want to dig a little deeper. Let's jump in!

## Prerequisites

To follow this tutorial, you will need the following:

- Access to a UNIX-like terminal environment, such as the terminal on Mac OS or Linux or Windows Subsystem for Linux.
- A working installation of [Docker Engine](https://docs.docker.com/engine/install/) or [Docker Desktop](https://docs.docker.com/desktop/).
- The [Grype scanner](https://github.com/anchore/grype#installation).

On most systems, Grype can be installed using the following command:

```sh
curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sudo sh -s -- -b /usr/local/bin
```

## Dockerfile Build Using Official Python Image

First, let's create a Dockerfile that uses the official Python image as a base image. The application will print the text `Hello, Linky! 🐙` to the console.

Using your terminal, create a project folder for our build and change the working directory to that folder:

```sh
mkdir -p ~/python-hello && cd $_
```

Now let's create a Python script using a text editor. In this example, we'll use Nano, a console text editor available by default on many systems.

```sh
nano hello.py
```

Enter the following Python code:

```python
print("Hello, Linky! 🐙")
```

To save the file, press `Control-x`, then `y`, then hit `Enter`. This will save the file as `hello.py` in the current directly and close Nano.

Next, we'll need a Dockerfile:

```sh
nano Dockerfile
```

Enter the following text in the `Dockerfile`:

```Dockerfile
FROM python
COPY hello.py hello.py
ENTRYPOINT ["python", "hello.py"]
```

Save this text (`Control-x`, `y`, `Enter`).

Now let's build the image:

```sh
docker build . -t python-hello
```

Finally, run the image:

```sh
docker run python-hello
```

You should see the text `Hello, Linky! 🐙` as output. We've now containerized a one-line Python application using the official Python image.

## Scanning for CVEs with Grype

Now we'll use the Grype scanner to check this newly created Python image for vulnerabilities. For this exercise, we'll use the [Grype container image maintained by Chainguard](https://images.chainguard.dev/directory/image/grype/overview). However, you may want to consider [installing Grype](https://github.com/anchore/grype#installation) on your host machine, as it's a useful tool to have in your vulnerability management toolbox. 

Run the following command to scan the `python-hello` image with Grype:

```sh
grype python-hello
```

On running this command, you should see dynamic output from Grype as the official Python image is pulled and scanned. After some time, you should receive a large amount of output. Output from Grype is divided into two parts: an initial overview and an itemized list of specific vulnerabilities. Let's first take a look at the initial overview — you may need to scroll up to find it.

``` 
 ✔ Loaded image                                                                                                                          python-hello:latest
 ✔ Parsed image                                                                      sha256:371d06a727271e250c913fb6e3adade4ce8d1f1a85df2ff96f072e0f3dfa0406
 ✔ Cataloged contents                                                                       4abedac2a866bcac0a507d433f2f2ba21fa8d1466be068ffcbf022d747728877
   ├── ✔ Packages                        [438 packages]  
   ├── ✔ File digests                    [20,064 files]  
   ├── ✔ File metadata                   [20,064 locations]  
   └── ✔ Executables                     [1,425 executables]  
 ✔ Scanned for vulnerabilities     [1276 vulnerability matches]  
   ├── by severity: 7 critical, 118 high, 325 medium, 53 low, 612 negligible (161 unknown)
   └── by status:   75 fixed, 1201 not-fixed, 541 ignored 
```

Arguably the most important line in this output is the breakdown of vulnerabilities by severity. CVEs are assigned numerical scores according to the [Common Vulnerability Scoring System (CVSS)](https://nvd.nist.gov/vuln-metrics/cvss). These scores correspond to four categories:

- Critical (9.0-10.0)
- High (7.0-8.9)
- Medium (4.0-6.9)
- Low (0.1-3.9)

The "by severity" line in our Grype output suggests that, as of February 24th, 2025, the official Python image had 7 critical, 118 high, 325 medium, and 53 low vulnerabilities.

``` 
   ├── by severity: 7 critical, 118 high, 325 medium, 53 low, 612 negligible (161 unknown)
```

Another line to look out for is how many of the CVEs have been fixed in a release:

``` language-markup
   └── by status:   75 fixed, 1201 not-fixed, 541 ignored 
```

If you're seeing CVEs with a "fixed" status, that indicates that updating to a new version of the affected package will resolve the issue. The more often an image is rebuilt with updated packages, the fewer fixed CVEs will be found.

Grype also itemizes each CVE found during the scan. This constitutes the second half of our output. Let's rerun the scan to look just at the CVE that have critical status:

``` 
grype python-hello | grep -i critical 
```

This produces the following output:

```
n't fix)        deb     CVE-2023-6879     Critical    
libopenexr-3-1-30             3.1.5-5                       (won't fix)        deb     CVE-2023-5841     Critical    
libopenexr-dev                3.1.5-5                       (won't fix)        deb     CVE-2023-5841     Critical    
wget                          1.21.3-1+b2                   (won't fix)        deb     CVE-2024-38428    Critical    
zlib1g                        1:1.2.13.dfsg-1               (won't fix)        deb     CVE-2023-45853    Critical    
zlib1g-dev                    1:1.2.13.dfsg-1               (won't fix)        deb     CVE-2023-45853    Critical
```

Note that Grype claimed 7 critical CVEs in the summary, but there are 6 in this output. Grype will remove itemized CVEs if they represent a package installed twice in different locations on the system. This can be confusing, but is the tool's intended behavior.

If you're resolving CVEs manually, this output is your starting point. Generally, you would look up the CVE on[ nist.gov](https://www.nist.gov/), which collects information on affected packages and systems, disclosures, and additional resources for learning about CVEs, such as advisories and potential mitigations. Since the above packages are marked as (`won't fix)`, you will need to either determine that the CVE isn't relevant to your use case, remove the affected package, or manually mitigate the CVE.


## Migrating to Chainguard Images

We've seen that using the official Python image as a base image for our application leads to a build with many known vulnerabilities. Let's switch to the Chainguard Python Image as our base image by changing our `Dockerfile`.

```sh
nano Dockerfile
```

Edit the `FROM` line to pull from Chainguard's public Python image:

```
FROM cgr.dev/chainguard/python
```

Your updated `Dockerfile` should look as follows:

```Dockerfile
FROM cgr.dev/chainguard/python
COPY hello.py hello.py
ENTRYPOINT ["python", "hello.py"]
```

Build the Chainguard version of the image, tagging it `chainguard-hello`:

```sh
docker build . -t chainguard-hello
```

Now run the image:

```sh
docker run chainguard-hello
```

You should see output as before: `Hello, Linky! 🐙`. Let's scan the new build using Grype:

```sh
grype chainguard-hello
```

On most days this scan is run, you will find zero CVEs in the Python Chainguard Image. On February 24, 2025, these is one CVE reported:

```
python-3.13  3.13.2-r2            apk   CVE-2024-3220  Unknown
```

Unknown CVEs are CVEs that are currently under investigation. They may or may not be assigned a severity status in the future. In this case, we might consider this a 99.9986% reduction in CVEs (from 736 to 1), though an analysis that more properly ignored CVEs with unknown status would put this at a 100% reduction. 😎

## Resources

- [Overview of Chainguard Images](https://edu.chainguard.dev/chainguard/chainguard-images/overview/)
- [Migrating to Python Chainguard Images](https://edu.chainguard.dev/chainguard/migration/migrating-python/)
- [Updating a Python Microservice for Chainguard Images](https://edu.chainguard.dev/chainguard/migration/porting-apps-to-chainguard/#updating-the-python-microservice)
- [Blog Post: Securely Containerize a Python Application with Chainguard Images](https://dev.to/chainguard/securely-containerize-a-python-application-with-chainguard-images-bn8)
- [Video: How to containerize a Python application with a multi-stage build using Chainguard Images](https://www.youtube.com/watch?v=2D0JULd4E5A)
- [Learning Lab: Deploying a Flask App with Python and nginx Chainguard Images](https://www.youtube.com/watch?v=i6bDKplnp6I)

