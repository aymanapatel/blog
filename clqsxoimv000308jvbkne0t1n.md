---
title: "The quest to improve Supply Chain Security"
seoTitle: "The quest to improve Supply Chain Security. SBOM, SCA, SAST, VEX"
seoDescription: "The quest to improve Supply Chain Security. SBOM, SCA, SAST, DAST, VEX, CSAF"
datePublished: Sun Dec 31 2023 03:30:10 GMT+0000 (Coordinated Universal Time)
cuid: clqsxoimv000308jvbkne0t1n
slug: the-quest-to-improve-supply-chain-security
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1703965185274/24887bc4-a371-4944-9b68-04c9688441fe.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1703965228018/42d1503d-d9a7-4657-ac49-46ceb0850b45.png
tags: security, security-testing, supply-chain-management, securityawareness

---

# Introduction with a sour taste

Writing software is hard. Maintaining is harder. Securing it is the hardest. The attack vectors keep on increasing, year on year, as and when new features introduced. For instance, Log4j vulnerability that was seen a couple of years ago in December 2021, was because of a JNDI interface that was introduced in 2014. ([Blackhat attack talk](https://www.youtube.com/watch?v=Y8a5nB-vy78) on the exploitability in 2016!) Another example would be that of **Panama Papers leak**, where the open-source CMS Drupal was the [root cause of the hack.](https://www.prometsource.com/blog/drupal-security-updates-and-panama-papers)

Third example which nails down is the **Equifax** hack; whose root cause of not updating their Apache Struts vulnerability for [6 months](https://www.bleepingcomputer.com/news/security/equifax-confirms-hackers-used-apache-struts-vulnerability-to-breach-its-servers/) after a 0-day exploit was found.

Recent high-profile hacks which happened after the pandemic on Colonial pipelines, Microsoft Exchange server, Log4j, SolarWinds Hack; has made the people and governments weary of the security of software systems. Government has come up with legislation in order to curb these hacks.

# Application Testing Tools

## SAST

> Static Application Security Testing

The security of application is determined by scanning static code. It does not have any information on the running application which can lead to some false-positives.

## DAST

> Dynamic Application Security Testing

The security of application is determined by running test on the running application. This can be achieved by sending malicious code/input fields . As it does not look into the code, it has not context on root cause of a security vulnerability. Hence, it will be difficult for developer to understand and guage what architectural/code issues are the root cause of the security vulnerability.

## IAST

> Interactive Application Security Testing

The security of application is determined by running an agent inside the application similar to agents deployed for monitoring such as Dynatace, eBPF etc It has advantages of both DAST and SAST. It can run against deployed application as well as see the source code.

Following illustration is on what DAST, SAST, IAST tests:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703997703028/a2965239-b587-4ed8-bcda-f5d5d5746f5a.jpeg align="center")

## SCA

> Software Component Analysis

This provides information on dependencies or libraries. Information include

1. Downstream vulnerabilities
    
2. License risks
    
3. Library health (how well maintained is the library)
    

SCA's provide an interface with dashboards on number of issues (license, vulnerability and package health); and also provide links to CVEs which can aid developers and security engineers to get all information of the software dependencies at a single place.

| Type | Example | What | Cons | Pros |
| --- | --- | --- | --- | --- |
| SAST | Checkmax, Sonarqube | Look at code to find vulnerability | False positives possible | Find exact security issue in the client code |
| IAST | Invicti, Semgrep, Contrast security | Interactive testing by embedded to running application and running security test against application | Requires agent to run which may not be available for a particular language | Ability to see source code like SAST while running real security attacks like DAST |
| DAST | Bright security, Veracode DAST | Run security tests from outside | Developers cannot grt root cause of vulnerablity.Requires additional time root causing and fixing them. | Fast to run in CI/CD pipeline |
| SCA | Synk, Blackduck | Look at vulnerabilities of libraries. Other things is also to consider licenses of libraries as well as how well-maintained | Very low Signal-to-noise ratio | Dashboard provides detals on security, package healt at one place |

The following caability matrix by veracode provides the capabilities f each tool:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703996759226/6b5ac010-30c2-448b-b13a-8d013fe48473.jpeg align="center")

# SBOM, VEX and CSAF

According to US ruling, it is mandated for companies to include a software-bill-of-materials. Generally, bill of materials is traditionally a list of items that was used to build a particular product. For example, you might have heard of card recalls due to some fault at brake discs. BOM allows for auto-manufacturers to pinpoint the raw material to the cars where it was used, so that they can trace from raw materials to the cars where the materials were used. In software also, there are standards to define the components of software, which are mainly libraries.

SCA might sound similar to SBOM; but goal of SBOM is to collaborate with other systems. SCA is vendor-specific, but SBOMs are driven by foundations such as OWASP and Linux Foundation, which provides interoperability for further things such as advisory framework, exploitability of vulnerability as well as signing of software packages to maintain authenticity of the packages.

## SBOM

There are 2 standards for SBOM

* SPDX (by Linux foundation)
    
* CylconeDX (by OWASP)
    

### SPDX

This was created by Linux foundation in 2011 to track software licenses. After some years, information regarding the materials/components of software was added.

### CycloneDX

Cyclone DX was created by OWASP for combating vulnerability identification, outdate softwares and license compliance. It not only included SBOM but also [HBOM](https://github.com/CycloneDX/bom-examples/blob/master/HBOM) (Hardware), [OBOM](https://github.com/CycloneDX/bom-examples/blob/master/SBOM) (Operations), [SaaSBOM](https://github.com/CycloneDX/bom-examples/blob/master/SaaSBOM) (Software as a Service), [VDR](https://github.com/CycloneDX/bom-examples/blob/master/VDR) (Vulnerability Disclosure Report), [VEX](https://github.com/CycloneDX/bom-examples/blob/master/VEX) (Vulnerability Explotability eXchange). VDR and VEX is more important and useful when used with SBOM for software engineers in order to find vulnerabilities, create a report on impact through advisory and finally making a decision if vulnerability is explitable in the current software stack.

![CycloneDX Object Model Swimlane](https://cyclonedx.org/theme/assets/images/CycloneDX-Object-Model-Swimlane.svg align="left")

## SBOM Tools

1. Anchore/Syft
    

Generate SBOM for your container images, libraries, filesystems

a. Generating CyclineDX and SPDX using [Syft](https://github.com/anchore/syft)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703964005902/ef98829e-4c13-4a69-86d3-b0b613b13c29.jpeg align="left")

b. JSON output from SPDX and Cyclone DX

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703964009522/5d8e734e-be48-45a2-83cc-9445ade086ba.jpeg align="left")

1. Anchore/Grype Works with Syft and does vulnerability scanning for containers as well as filesystems.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703964019222/75b7dc54-2ecb-49e2-99a2-fb9a64475ab5.jpeg align="left")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703964022658/20ca64ea-14c9-473a-86de-3d47a0ffb977.jpeg align="left")

1. [Synk](https://snyk.io) provides all tools under 1 hood. It provides tools such as
    

* Synk OpenSource: Identify vulnerable open source libraries
    
* Synk Container Image: Securing container images
    
* Snyk IaC: Misconfiguration of IaC secrets and policies
    
* Snyk Code: SAST offering form Snyk
    

1. Aqua
    

* Aqua vSheild: Mitigation strategy for vulnerability
    
* Aqua Trivy: OSS Vulnerability scanner, IaC protection
    

## VEX (Vulnerability eXploit)

Goal of VEX is to provide information on the \*\*to communicate the exploitability of components with known vulnerabilities in the context of the product in which they are used."

Having a lot of libraries can lead to a lot of noise if there is a vulnerability. The vulnerability can be an obscure function which is hardly used. So in order to combat such scenarios, where we need clarity on the risk of the vulnerability so that appropriate action can be taken on fixing it or keeping it on hold for some time as it is not. arisk at this point of time.

Goals of VEX:

* VEX can be inserted into a BOM (CycloneDX); or have an dedicated VEX BOM.
    
* Provide ̑ clear details on vulnerability, exploitability.
    
* Bridge the communication gap between software consumer and software producer. It can communicate to software consumer on the actions taken by the software producer and actions to be taken by consumer to minimize security impact.
    

![[E192622E-215F-445A-978B-4FBF76C1C020.jpeg]]( align="left")

## CSAF

Common Security Advisory Framework (CSAF) is a language to communicate security advisories. It is the final step of understanding the software components, and providing enough information in order to make the decision of remediating software vulnerabilities.

It is maintained by OASIS Open and a CSAF v2.0 standard can be found [here](https://docs.oasis-open.org/csaf/csaf/v2.0/os/csaf-v2.0-os.html)

Example CSAF JSON

```json
{
  "document": {
    "category": "csaf_vex",
    "csaf_version": "2.0",
    "notes": [
      {
        "category": "summary",
        "text": "Example Company VEX document. Unofficial content for demonstration purposes only.",
        "title": "Author comment"
      }
    ],
    "publisher": {
      "category": "vendor",
      "name": "Example Company ProductCERT",
      "namespace": "https://psirt.example.com"
    },
    "title": "Example VEX Document Use Case 1 - Fixed",
    "tracking": {
      "current_release_date": "2022-03-03T11:00:00.000Z",
      "generator": {
        "date": "2022-03-03T11:00:00.000Z",
        "engine": {
          "name": "Secvisogram",
          "version": "1.11.0"
        }
      },
      "id": "2022-EVD-UC-01-F-001",
      "initial_release_date": "2022-03-03T11:00:00.000Z",
      "revision_history": [
        {
          "date": "2022-03-03T11:00:00.000Z",
          "number": "1",
          "summary": "Initial version."
        }
      ],
      "status": "final",
      "version": "1"
    }
  },
  "product_tree": {
    "branches": [
      {
        "branches": [
          {
            "branches": [
              {
                "category": "product_version",
                "name": "1.1",
                "product": {
                  "name": "Example Company DEF 1.1",
                  "product_id": "CSAFPID-0001"
                }
              }
            ],
            "category": "product_name",
            "name": "DEF"
          }
        ],
        "category": "vendor",
        "name": "Example Company"
      }
    ]
  },
  "vulnerabilities": [
    {
      "cve": "CVE-2021-44228",
      "notes": [
        {
          "category": "description",
          "text": "Apache Log4j2 2.0-beta9 through 2.15.0 (excluding security releases 2.12.2, 2.12.3, and 2.3.1) JNDI features used in configuration, log messages, and parameters do not protect against attacker controlled LDAP and other JNDI related endpoints. An attacker who can control log messages or log message parameters can execute arbitrary code loaded from LDAP servers when message lookup substitution is enabled. From log4j 2.15.0, this behavior has been disabled by default. From version 2.16.0 (along with 2.12.2, 2.12.3, and 2.3.1), this functionality has been completely removed. Note that this vulnerability is specific to log4j-core and does not affect log4net, log4cxx, or other Apache Logging Services projects.",
          "title": "CVE description"
        }
      ],
      "product_status": {
        "fixed": [
          "CSAFPID-0001"
        ]
      }
    }
  ]
}
```

# Signing your software libraries

There are a couple of attack vectors that are based on how software is installed. A couple of years ago, security researcher wrote this article which went viral ["Dependency Confusion: How I Hacked Into Apple, Microsoft and Dozens of Other Companies"](https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610). He was able to download private repos from companies. Another example is series of attacks on npm where a misnomer(spelling mistake) of packages led to installation of malware. These sort of attacks could have been mitigated by using digital signatures.

We have encryption which follows the practice of integrity, authenticity and non-repudiation. Software libraries signing can allow us to keep malicious packages out of our systems. There are a number of technologies which are using the signing principles of cryptography but for software binaries/libraries/containers signing.

## Sigstore

A standard and a collection of tools for enabling software components to be cryptographically digitally signed.

It includes:

1. Cosign: Signing software artifalcts
    
2. Fulcio: Providing Certificate Authority
    
3. Rekor: Transparency log
    
4. Gitsign: Signing Git commits
    
5. OpenID integration: Authentication to check identity of requestor
    

![Lifecycle of sigstore](https://www.sigstore.dev/img/alt_landscapelayout_overview.svg align="left")

Can see the demo of sigstore [here](https://www.youtube.com/watch?v=G7gU3WZTBpk)

# Resources

1. [Infinite CVEs with Supply chain](https://www.cramhacks.com/p/INFINITE-CVES-WITH-SUPPLY-CHAIN)
    
2. Semgrep
    
    1. [Supply chain security is hard](https://semgrep.dev/blog/2022/software-supply-chain-security-is-hard/)
        
    2. [Best supply chain security tool - The lock file](https://semgrep.dev/blog/2022/the-best-free-open-source-supply-chain-tool-the-lockfile/)
        
3. [OSSF Vulnerability disclosure](https://github.com/ossf/wg-vulnerability-disclosures)