---
title: Install CockroachDB on Mac
summary: Install CockroachDB on Mac, Linux, or Windows. Sign up for product release notes.
tags: download, binary, homebrew
toc: true
key: install-cockroachdb.html
docs_area: deploy
---

<div id="os-tabs" class="clearfix">
  <button id="mac" class="current" data-eventcategory="buttonClick-doc-os" data-eventaction="mac">Mac</button>
  <a href="install-cockroachdb-linux.html"><button id="linux" data-eventcategory="buttonClick-doc-os" data-eventaction="linux">Linux</button></a>
  <a href="install-cockroachdb-windows.html"><button id="windows" data-eventcategory="buttonClick-doc-os" data-eventaction="windows">Windows</button></a>
</div>

To find out what's new in the latest release, {{ page.release_info.version }}, refer to [Release Notes](../releases/{{page.version.version}}.html). To upgrade to this release from an earlier version, refer to [Cluster Upgrade](upgrade-cockroach-version.html).

{% include cockroachcloud/use-cockroachcloud-instead.md %}

Use one of the following options to install CockroachDB.

<a id="use-homebrew"></a>

## Install using Homebrew

Homebrew installs binaries for your system architecture, either Intel or ARM ([Apple Silicon](https://support.apple.com/en-us/HT211814)).

{{site.data.alerts.callout_info}}
CockroachDB's [spatial features](spatial-features.html) are not available on ARM systems when CockroachDB is installed using Homebrew. To use these features, install CockroachDB using one of these methods instead of using Homebrew:

- Use [Rosetta](https://developer.apple.com/documentation/virtualization/running_intel_binaries_in_linux_vms_with_rosetta) to [install the Intel macOS binary](#install-binary) of CockroachDB.
- Run CockroachDB using the [Docker image](#use-docker). On macOS, Docker images run in a lightweight Linux virtual machine. The GEOS libraries are automatically included and configured within the Docker image.
{{site.data.alerts.end}}

1. [Install Homebrew](http://brew.sh/).
1. If you previously installed CockroachDB using Homebrew, use Homebrew to uninstall it before installing a new version:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    brew uninstall cockroach
    ~~~

    If you installed CockroachDB using a different method, you may need to remove the binary before installing using Homebrew.

1. Use Homebrew to install CockroachDB:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    brew install cockroachdb/tap/cockroach
    ~~~

1. Make sure the `cockroach` binary you just installed is the one that runs when you type `cockroach` in your shell:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    which cockroach
    ~~~

    ~~~
    /opt/homebrew/bin/cockroach
    ~~~

1. If you plan to use CockroachDB's [spatial features](spatial-features.html), refer to [Install GEOS libraries](#install-geos-libraries). Otherwise, your installation is complete.

1. Keep up-to-date with CockroachDB releases and best practices:

    {% include marketo-install.html uid="1" %}

<a id="download-the-binary"></a>
<a id="install-binary"></a>

## Install the binary

{{site.data.alerts.callout_info}}
CockroachDB's [spatial features](spatial-features.html) are disabled in the binary for ARM ([Apple Silicon](https://support.apple.com/en-us/HT211814)) systems. To use these features, do not install the ARM binary. Instead, you can:

- Use [Rosetta](https://developer.apple.com/documentation/virtualization/running_intel_binaries_in_linux_vms_with_rosetta) to [install the Intel macOS binary](#install-binary) of CockroachDB.
- Run CockroachDB using the [Docker image](#use-docker). On macOS, Docker images run in a lightweight Linux virtual machine. The GEOS libraries are automatically included and configured within the Docker image.
{{site.data.alerts.end}}

1. Visit [Releases](/docs/releases/index.html) to download the CockroachDB archive your system architecture, either Intel or ARM ([Apple Silicon](https://support.apple.com/en-us/HT211814)). The archive contains the `cockroach` binary and the supporting libraries that are used to provide [spatial features](spatial-features.html).

    You can download the binary using a web browser or you can copy the link and use a utility like `curl` to download it. If you download the ARM binary using a web browser and you plan to use CockroachDB's spatial features, an additional step is required before you can install the library, as outlined in the next step.

1. Extract the archive and optionally copy the `cockroach` into a directory in your `PATH` so you can execute [`cockroach` commands](cockroach-commands.html) from any shell. If you get a permission error, use `sudo` to copy the `cockroach` binary to a system directory.
1. Make sure the `cockroach` binary you just installed is the one that runs when you type `cockroach` in your shell:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    which cockroach
    ~~~

    ~~~
    /usr/local/bin/cockroach
    ~~~

1. If you plan to use CockroachDB's [spatial features](spatial-features.html), refer to [Install GEOS libraries](#install-geos-libraries). Otherwise, your installation is complete.
1. Keep up-to-date with CockroachDB releases and best practices:

    {% include marketo-install.html uid="2" %}

<a id="install-kubernetes"></a>

## Deploy using Kubernetes

To orchestrate CockroachDB locally using [Kubernetes](https://kubernetes.io/), refer to [Orchestrate CockroachDB Locally with Minikube](orchestrate-a-local-cluster-with-kubernetes.html).CockroachDB can be deployed using the [Helm](https://helm.sh/), package manager or a manual StatefulSet.


<a id="install-docker"></a>
<a id="use-docker"></a>

## Deploy using Docker

{{site.data.alerts.callout_danger}}
Running a stateful application like CockroachDB in Docker is more complex and error-prone than most uses of Docker. Unless you are very experienced with Docker, CockroachLabs recommends that you start with a different installation and deployment method.
{{site.data.alerts.end}}

CockroachDB Docker images are [multi-platform images](https://docs.docker.com/build/building/multi-platform/) that contains binaries for both Intel and ARM ([Apple Silicon](https://support.apple.com/en-us/HT211814)). CockroachDB on ARM systems is **experimental** and is not yet qualified for production use. Multi-platform images do not take up additional space on your Docker host.

Intel binaries can run on ARM systems, but with a significant reduction in performance.

1. Install [Docker for Mac](https://docs.docker.com/docker-for-mac/install/).">Docker for Mac</a>. For installation requirements and detailed instructions, refer to the Docker documentation.
1. Confirm that Docker is running in the background:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    docker version
    ~~~

    If an error occurs, confirm that Docker is running and try the command again.

1. Download the Docker image:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    docker pull {{page.release_info.docker_image}}:{{page.release_info.version}}
    ~~~

1. To start CockroachDB from Docker, refer to [Start a Cluster in Docker](start-a-local-cluster-in-docker-mac.html).

1. Keep up-to-date with CockroachDB releases and best practices:

    {% include marketo-install.html uid="3" %}


<a id="install-source"></a>

## Build from source

For guidance on building CockroachDB from source, refer [Getting and Building CockroachDB from Source](https://wiki.crdb.io/wiki/spaces/CRDB/pages/181338446/Getting+and+building+CockroachDB+from+source) in the CockroachDB Wiki.

<a id="install-geos-libraries"></a>

## Install the GEOS libraries (optional)

CockroachDB uses custom-built versions of the [GEOS libraries](spatial-glossary.html#geos). After you [install the CockroachDB binary](#download-the-binary), follow these steps to install the GEOS libraries.

{{site.data.alerts.callout_info}}
CockroachDB's [spatial features](spatial-features.html) are disabled in the binary for ARM ([Apple Silicon](https://support.apple.com/en-us/HT211814)) systems. To use these features, CockroachDB must be installed using one of these methods:

- Use [Rosetta](https://developer.apple.com/documentation/virtualization/running_intel_binaries_in_linux_vms_with_rosetta) to [install the Intel macOS binary](#install-binary) of CockroachDB.
- Run CockroachDB using the [Docker image](#use-docker). On macOS, Docker images run in a lightweight Linux virtual machine. The GEOS libraries are automatically included and configured within the Docker image.
{{site.data.alerts.end}}

1. Make sure the `cockroach` binary is in your shell's `PATH`, and make a note of its location.

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    which cockroach
    ~~~

    ~~~
    /usr/local/bin/cockroach
    ~~~

    In the following instructions, replace `{cockroach_directory}` with this location.

1. Go to the directory where you extracted the CockroachDB binary archive before you installed it. The GEOS libraries are located in the `lib/` subdirectory.
1. Copy the GEOS libraries to one of the following locations: <ul><li><code>/usr/local/lib/cockroach</code></li><li><code>{cockroach_directory}/lib</code></li><li>A directory of your choice</li></ul>

    If the GEOS libraries are installed in a different location, you must provide the location using the `--spatial-libs` flag to `cockroach start`.

    In the following instructions, replace `{geos_library_directory}` with the target directory for the GEOS libraries:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    cp lib/libgeos.dylib {geos_library_directory} && cp libgeos_c.dylib {geos_library_directory}
    ~~~

    If you get a permissions error, use `sudo` to copy each GEOS library to a system directory.

1. Verify that CockroachDB can execute spatial queries.

   1. Start a temporary, in-memory cluster using the [`cockroach demo`](cockroach-demo.html) command:

        {% include_cached copy-clipboard.html %}
        ~~~ shell
        cockroach demo
        ~~~

        The cluster starts and the interactive SQL runs, just like when you run the `cockroach sql` command.

   1. In interactive SQL shell, run the following query to test that CockroachDB supports spatial features:

        {% include_cached copy-clipboard.html %}
        ~~~ sql
        SELECT ST_IsValid(ST_MakePoint(1,2));
        ~~~

        If CockroachDB successfully found the GEOS libraries, the query returns the following output:

        ~~~ sql
        st_isvalid
        --------------
        true
        (1 row)
        ~~~

        If CockroachDB cannot find the GEOS libraries, the query returns an error:

        ~~~ sql
        ERROR: st_isvalid(): geos: error during GEOS init: geos: cannot load GEOS from dir "/usr/local/lib/cockroach": failed to execute dlopen
          Failed running "sql"
        ~~~

## What's Next?

{% include {{ page.version.version }}/misc/install-next-steps.html %}

{% include {{ page.version.version }}/misc/diagnostics-callout.html %}
