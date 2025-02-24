# docker scout cves

```
docker scout cves [OPTIONS] [IMAGE|DIRECTORY|ARCHIVE]
```

<!---MARKER_GEN_START-->
Display CVEs identified in a software artifact

### Options

| Name                   | Type          | Default    | Description                                                                                                                                                                                                                                                   |
|:-----------------------|:--------------|:-----------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--details`            |               |            | Print details on default text output                                                                                                                                                                                                                          |
| `--env`                | `string`      |            | Name of environment                                                                                                                                                                                                                                           |
| `-e`, `--exit-code`    |               |            | Return exit code '2' if vulnerabilities are detected                                                                                                                                                                                                          |
| `--format`             | `string`      | `packages` | Output format of the generated vulnerability report:<br>- packages: default output, plain text with vulnerabilities grouped by packages<br>- sarif: json Sarif output<br>- markdown: markdown output (including some html tags like collapsible sections)<br> |
| `--ignore-base`        |               |            | Filter out CVEs introduced from base image                                                                                                                                                                                                                    |
| `--locations`          |               |            | Print package locations including file paths and layer diff_id                                                                                                                                                                                                |
| `--multi-stage`        |               |            | Show packages from multi-stage Docker builds                                                                                                                                                                                                                  |
| `--only-cve-id`        | `stringSlice` |            | Comma separated list of CVE ids (like CVE-2021-45105) to search for                                                                                                                                                                                           |
| `--only-fixed`         |               |            | Filter to fixable CVEs                                                                                                                                                                                                                                        |
| `--only-package`       | `stringSlice` |            | Comma separated regular expressions to filter packages by                                                                                                                                                                                                     |
| `--only-package-type`  | `stringSlice` |            | Comma separated list of package types (like apk, deb, rpm, npm, pypi, golang, etc)                                                                                                                                                                            |
| `--only-severity`      | `stringSlice` |            | Comma separated list of severities (critical, high, medium, low, unspecified) to filter CVEs by                                                                                                                                                               |
| `--only-stage`         | `stringSlice` |            | Comma separated list of multi-stage Docker build stage names                                                                                                                                                                                                  |
| `--only-unfixed`       |               |            | Filter to unfixed CVEs                                                                                                                                                                                                                                        |
| `--only-vuln-packages` |               |            | When used with --format=only-packages ignore packages with no vulnerabilities                                                                                                                                                                                 |
| `--org`                | `string`      |            | Namespace of the Docker organization                                                                                                                                                                                                                          |
| `-o`, `--output`       | `string`      |            | Write the report to a file.                                                                                                                                                                                                                                   |
| `--platform`           | `string`      |            | Platform of image to analyze                                                                                                                                                                                                                                  |
| `--ref`                | `string`      |            | Reference to use if the provided tarball contains multiple references.<br>Can only be used with --type archive.                                                                                                                                               |
| `--type`               | `string`      | `image`    | Type of the image to analyze. Can be one of:<br>- image<br>- oci-dir<br>- archive (docker save tarball)<br>- fs (directory or file)<br>                                                                                                                       |
| `--vex`                |               |            | Apply VEX statements to filter CVEs                                                                                                                                                                                                                           |
| `--vex-author`         | `stringSlice` |            | List of VEX statement authors to accept                                                                                                                                                                                                                       |
| `--vex-location`       | `stringSlice` |            | File location of directory or file containing VEX statements                                                                                                                                                                                                  |


<!---MARKER_GEN_END-->

## Description

The `docker scout cves` command analyzes a software artifact for vulnerabilities.

If no image is specified, the most recently built image will be used.

The following artifact types are supported:

- Images
- OCI layout directories
- Tarball archives, as created by `docker save`

The tool analyzes the provided software artifact, and generates a vulnerability report.

By default, the tool expects an image reference, such as:

- `redis`
- `curlimages/curl:7.87.0`
- `mcr.microsoft.com/dotnet/runtime:7.0`

If the artifact you want to analyze is an OCI directory or a tarball archive, you must use the `--type` flag.

## Examples

### Display vulnerabilities grouped by package

```console
$ docker scout cves alpine
Analyzing image alpine
    ✓ Image stored for indexing
    ✓ Indexed 18 packages
    ✓ No vulnerable package detected
```

### Display vulnerabilities from a `docker save` tarball

```console
$ docker save alpine > alpine.tar

$ docker scout cves --type archive alpine.tar
Analyzing archive alpine.tar
    ✓ Archive read
    ✓ SBOM of image already cached, 18 packages indexed
    ✓ No vulnerable package detected
```

### Display vulnerabilities from an OCI directory

```console
$ skopeo copy --override-os linux docker://alpine oci:alpine

$ docker scout cves --type oci-dir alpine
Analyzing OCI directory alpine
    ✓ OCI directory read
    ✓ Image stored for indexing
    ✓ Indexed 19 packages
    ✓ No vulnerable package detected
```

### Export vulnerabilities to a SARIF JSON file

```console
$ docker scout cves --format sarif --output alpine.sarif.json alpine
Analyzing image alpine
    ✓ SBOM of image already cached, 18 packages indexed
    ✓ No vulnerable package detected
    ✓ Report written to alpine.sarif.json
```

### Display markdown output

The markdown output also contains HTML tags to have a better rendering. This output can be used for instance in Pull Request comments.

```console
$ docker scout cves --format markdown alpine
    ✓ Pulled
    ✓ SBOM of image already cached, 19 packages indexed
    ✗ Detected 1 vulnerable package with 3 vulnerabilities
<h2>:mag: Vulnerabilities of <code>alpine</code></h2>

<details open="true"><summary>:package: Image Reference</strong> <code>alpine</code></summary>
<table>
<tr><td>digest</td><td><code>sha256:e3bd82196e98898cae9fe7fbfd6e2436530485974dc4fb3b7ddb69134eda2407</code></td><tr><tr><td>vulnerabilities</td><td><img alt="critical: 0" src="https://img.shields.io/badge/critical-0-lightgrey"/> <img alt="high: 0" src="https://img.shields.io/badge/high-0-lightgrey"/> <img alt="medium: 2" src="https://img.shields.io/badge/medium-2-fbb552"/> <img alt="low: 0" src="https://img.shields.io/badge/low-0-lightgrey"/> <img alt="unspecified: 1" src="https://img.shields.io/badge/unspecified-1-lightgrey"/></td></tr>
<tr><td>platform</td><td>linux/arm64</td></tr>
<tr><td>size</td><td>3.3 MB</td></tr>
<tr><td>packages</td><td>19</td></tr>
</table>
</details></table>
</details>
...
```

### List all packages of a certain typethat are vulnerable

The output will show the list of the packages of the image, that can be filtered, with the summary of vulnerabilities for each.

By default even packages with no vulnerabilities will be displayed.

```console
$ docker scout cves --format only-packages --only-package-type golang --only-vuln-packages golang:1.18.0
    ✓ Pulled
    ✓ SBOM of image already cached, 296 packages indexed
    ✗ Detected 1 vulnerable package with 40 vulnerabilities

   Name   Version   Type         Vulnerabilities
───────────────────────────────────────────────────────────
  stdlib  1.18     golang     2C    29H     8M     1L
```
