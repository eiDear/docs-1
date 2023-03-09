---
title: Install a FIPS-compliant build of CockroachDB
summary: Learn how to ensure that CockroachDB runs in an environment that enforces compliance with FIPS 140-2.
toc: true
docs_area: deploy
---

## Overview of FIPS 140-2 compliance in CockroachDB

{{site.data.alerts.callout_info}}
FIPS-compliant builds of CockroachDB are **experimental** and are not yet qualified for production use.
{{site.data.alerts.end}}

[Federal Information Processing Standards (FIPS) 140-2](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.140-2.pdf) is a mandatory standard for the protection of sensitive or valuable data within systems used by and for United States and Canadian federal government agencies. In the United States, the standard is applicable to all federal agencies that use cryptographic-based security systems to protect sensitive information in computer and telecommunication systems (including voice systems) as defined in Section 5131 of the Information Technology Management Reform Act of 1996, Public Law 104-106 and the Federal Information Security Management Act of 2002, Public Law 107-347.

When a cryptographic module or library has a FIPS 140-2 certificate, it has been tested and formally validated by the U.S. and Canadian Governments.

The standard CockroachDB binaries rely on the Go language for cryptographic functions and do not use OpenSSL libraries installed in the operating system. Go's cryptographic libraries are not FIPS-validated.

FIPS-compliant CockroachDB binaries are built using [Red Hat's Go FIPS with OpenSSL toolchain](https://github.com/golang-fips/go), which contains the necessary modifications for the Go crypto library to use an external cryptographic library in a FIPS compliant way. FIPS-compliant CockroachDB runtimes use the host operating system's OpenSSL libraries and commands, rather than Go's cryptographic libraries. When the installed OpenSSL has a FIPS 140-2 certificate and [FIPS mode is enabled in the Linux kernel](#enable-fips-mode-in-the-linux-kernel), the CockroachDB runtime is compliant with FIPS 140-2. FIPS-compliant binaries and Docker images are available for CockroachDB v22.2.7 and above. CockroachDB FIPS-compliant runtimes run only on Intel 64-bit Linux systems.

### FIPS-compliant features

When you use a FIPS-compliant CockroachDB runtime and each cluster node's OpenSSL is FIPS-validated and configured correctly, the following features have been tested to ensure that their cryptographic operations comply with FIPS 140-2:

- [Encryption At Rest](security-reference/encryption.html#encryption-keys-used-by-cockroachdb-self-hosted-clusters)
- [Encrypted Backups](take-and-restore-encrypted-backups.html)
- [Change Data Capture to Kafka over TLS](create-changefeed.html#query-parameters)
- [Certificate-based Node-to-Node and Client-to-Node Authentication](authentication.html)
- [SASL SCRAM-SHA-256 Password Authentication](security-reference/scram-authentication.html)
- [Built-in SQL cryptographic functions](functions-and-operators.html#cryptographic-functions)

  {{site.data.alerts.callout_danger}}
  In a FIPS-compliant system, Cockroach Labs recommends that you avoid using cryptographic operations that are not supported by FIPS 140-2. For example, generating an MD5 hash is not FIPS-compliant, because MD5 is not a FIPS-compliant algorithm. Use algorithms and functions that do not comply with FIPS at your own risk.
  {{site.data.alerts.end}}

This page shows how to configure {{ site.data.products.core }} to comply with FIPS 140-2 using Red Hat's FIPS-validated OpenSSL package. The FIPS-validated Docker image for CockroachDB comes with Red Hat's OpenSSL libraries pre-installed and configured.

### Performance considerations

When comparing performance of the same workload in a FIPS-compliant CockroachDB runtime to a standard CockroachDB runtime, some performance degradation is expected, due to the use of the system's installed FIPS-validated OpenSSL libraries. The amount of performance impact depends upon the workload, cluster configuration, query load, and other factors.

### Upgrading to a FIPS-compliant CockroachDB runtime

Upgrading an existing CockroachDB cluster's binaries in-place to be FIPS-compliant is not supported.

## Enable FIPS mode in the Linux kernel

The following section provides several ways to ensure that your system's OpenSSL libraries include the FIPS object module, and that functionality that is incompatible with FIPS 140-2 is disabled. The FIPS-compliant CockroachDB Docker images are based on [Red Hat's Universal Base Image 8 Docker image](https://catalog.redhat.com/software/containers/ubi8/ubi/5c359854d70cc534b3a3784e), which comes with FIPS-validated OpenSSL libraries pre-installed and is automatically compliant with FIPS 140-2. To [use the FIPS-compliant CockroachDB Docker image](#use-the-fips-compliant-cockroachdb-docker-image), you can skip directly to that section of this page.

{{site.data.alerts.callout_info}}
A [Ubuntu Pro or Ubuntu Advantage](https://ubuntu.com/pro) subscription is required to access [FIPS-validated OpenSSL libraries and pre-configured Docker images](https://ubuntu.com/security/certifications/docs/fips). See [Enabling FIPS with the `ua` tool](https://ubuntu.com/security/certifications/docs/fips-enablement) in Ubuntu's documentation. This page does not provide specific instructions for Ubuntu.
{{site.data.alerts.end}}

A system must have FIPS mode enabled in the kernel before it can run the FIPS-compliant CockroachDB binary or that starts containers that run the FIPS-compliant CockroachDB Docker image. When FIPS mode is enabled, the FIPS kernel module is loaded and cryptographic operations in the kernel comply with FIPS 140-2. To enable FIPS mode on Red Hat Enterprise Linux or its derivatives, refer to [Enable FIPS mode](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/security_hardening/assembly_installing-a-rhel-8-system-with-fips-mode-enabled_security-hardening) in Red Hat's documentation. To verify that FIPS mode is enabled, refer to [Verify that the kernel enforces FIPS compliance](#verify-that-the-kernel-enforces-fips-compliance).

### Extend Red Hat's Universal Base Image 8 Docker image

If you do not want to use the FIPS-compliant CockroachDB Docker image directly, you can create a custom Docker image based on [Red Hat's Universal Base Image 8 Docker image](https://catalog.redhat.com/software/containers/ubi8/ubi/).

- You can model your Dockerfile on the one that Cockroach Labs uses to produce the [FIPS-compatible Docker image](https://github.com/cockroachdb/cockroach/blob/master/build/deploy/Dockerfile) for CockroachDB.
- Your Dockerfile must install OpenSSL before it starts the `cockroach` binary.
- FIPS mode must be enabled on the Docker host kernel before it can run containers with FIPS mode enabled. The FIPS-compliant CockroachDB Docker image must run with FIPS mode enabled. To enable FIPS mode in the Docker host kernel, refer to [Enable FIPS mode](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/security_hardening/assembly_installing-a-rhel-8-system-with-fips-mode-enabled_security-hardening) in Red Hat's documentation. To verify that FIPS mode is enabled, refer to [Verify that the kernel enforces FIPS compliance](#verify-that-the-kernel-enforces-fips-compliance).

### Install on Red Hat Enterprise Linux

When you install Red Hat Enterprise Linux (RHEL) on a host, or after installation has completed, you can [enable FIPS mode](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/security_hardening/assembly_installing-a-rhel-8-system-with-fips-mode-enabled_security-hardening). Red Hat recommends installing RHEL with FIPS mode enabled, as opposed to enabling FIPS mode later. Enabling FIPS mode during the installation ensures that the system generates all keys with FIPS-approved algorithms and continuous monitoring tests in place.

After FIPS mode is enabled, when you install OpenSSL, FIPS-validated OpenSSL libraries are installed. To install OpenSSL:

{% include_cached copy-clipboard.html %}
~~~ shell
RUN microdnf -y install openssl
~~~

Next, [verify that the kernel enforces FIPS compliance](#verify-that-the-kernel-enforces-fips-compliance).

### Verify that the kernel enforces FIPS compliance

If FIPS mode is not enabled in the Linux kernel, or if OpenSSL is not configured correctly, it does not enforce FIPS 140-2 by default even if the FIPS module is present. Take these steps to verify that the Linux kernel is enforcing FIPS 140-2:

1. You can list the FIPS-validated ciphers using the `openssl ciphers` command. If FIPS mode is disabled, a `no cipher match` error occurs.

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    openssl ciphers FIPS -v
    ~~~

1. When FIPS mode is enabled, the `sysctl` variable `fips` is set to `1`:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    sudo sysctl -a|grep fips
    ~~~

    ~~~ shell
    fips=1
    ~~~

    {{site.data.alerts.callout_info}}
    This command does not work from within a Docker container. A Docker container runs with FIPS mode enabled if the Docker host has FIPS mode enabled. Run this command from the Docker host rather than the Docker container.
    {{site.data.alerts.end}}

1. The MD5 hashing algorithm is not compatible with FIPS 140-2. If the following command **succeeds**, FIPS mode is not enabled:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    echo "test" | openssl md5
    ~~~

After verifying that the kernel enforces FIPS compliance, you can [install the FIPS-compliant CockroachDB binary](#install-the-fips-compliant-cockroachdb-binary). If the tests indicate that FIPS compliance is not enforced, refer to your operating system's documentation about enabling FIPS mode to resolve the issue before continuing.

## Install the FIPS-compliant CockroachDB binary

To install the FIPS-compliant CockroachDB binary:

1. Visit [Releases](/docs/releases/index.html) and download the FIPS-compliant CockroachDB binary for Intel 64-bit Linux. FIPS-compliant CockroachDB binaries are available for v23.1.0-alpha.5 and above.
1. [Install the binary](install-cockroachdb-linux.html) that you downloaded.

{{site.data.alerts.callout_info}}
Upgrading an existing CockroachDB cluster's binaries in-place to be FIPS-compliant is not supported.
{{site.data.alerts.end}}

### Verify that CockroachDB is FIPS-compliant

If the CockroachDB binary is FIPS-compliant, the string `fips` is appended to the Go version in the `cockroach version` command:

{% include_cached copy-clipboard.html %}
~~~ shell
cockroach version |grep fips
~~~

~~~ shell
go1.19.5fips
~~~

This indicates that CockroachDB was built using [Red Hat's Go FIPS with OpenSSL toolchain](https://github.com/golang-fips/go).

## Use the FIPS-compliant CockroachDB Docker image

1. If necessary, enable FIPS mode on the Docker host. The FIPS-compliant CockroachDB Docker image must run with FIPS mode enabled. To enable FIPS mode in the Docker host kernel, refer to [Enable FIPS mode](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/security_hardening/assembly_installing-a-rhel-8-system-with-fips-mode-enabled_security-hardening) in Red Hat's documentation. To verify that FIPS mode is enabled, refer to [Verify that the kernel enforces FIPS compliance](#verify-that-the-kernel-enforces-fips-compliance).
1. Visit [Releases](/docs/releases/index.html?filters=docker) and copy the name of a FIPS-compliant Docker image tag.
1. Pull the Docker image locally, create a new container that uses it, run the container, and attach to it. This example gives the running container the name `cockroachdb-fips-container`. Replace `<image_tag>` with the Docker image tag you copied.

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    docker run <image_tag> --name="cockroachdb-fips-container" -i
    ~~~

1. In the running container, [verify that OpenSSL enforces FIPS compliance](#verify-that-the-kernel-enforces-fips-compliance).

    {{site.data.alerts.callout_info}}
    Do not attempt to run the `sysctl` test from within the running container.
    {{site.data.alerts.end}}

1. In the running container, [verify that CockroachDB is FIPS-compliant](#verify-that-cockroachdb-is-fips-compliant).
1. To stop the container, use `CTRL-C`. To detach from the container but keep it running in the background, use the sequence `CTRL+P+CTRL+Q`.

## See also

- [Install CockroachDB](install-cockroachdb-linux.html)
- [Releases](/docs/releases/index.html)
