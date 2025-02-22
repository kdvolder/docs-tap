# tanzu insight source add

This topic tells you how to use the Tanzu Insight CLI 
`tanzu insight source add` command to add a source report.

## <a id='synopsis'></a>Synopsis

Add a source to the metadata store with the source's SBOM or vulnerability scan. By default command will show Standard output and errors together. Output and errors can be redirected to files by using 1>out.txt 2>error.txt

```console
tanzu insight source add --path <filepath> [flags]
```

## <a id='examples'></a>Examples

```console
insight source add --input-format cyclonedx-xml --output-format text --path /path/to/file.xml
  insight source add --input-format spdx-json --output-format api-json --path /path/to/file.json
  insight source add --path /path/to/file.xml
  insight source add --path /path/to/file.xml --artifact-group-uid workload-uid --artifact-group-label name=example --artifact-group-label namespace=example-ns
  insight source add --path /path/to/file.xml --artifact-group-uid workload-uid --artifact-group-label name=example,namespace=example-ns
  insight source add --input-format spdx-json --output-format api-json --path /path/to/file.json 1>out.txt 2>error.txt (* Output will be redirected to out.txt and error will be redirected to error.txt .)
```

## <a id='options'></a>Options

```console
      --artifact-group-label strings   specify artifact group labels, must be in key=value format. This flag can be set multiple times to provide multiple labels in one call, or can be passed a list of labels separated by commas. Note: this flag can only be specified if artifact-group-uid flag is set
      --artifact-group-uid string      uid of artifact group to add to, or create if it doesn't already exist
  -h, --help                           help for add
  -i, --input-format string            specify the file’s SBOM report format and file type, options=[cyclonedx-xml, cyclonedx-json, spdx-json] (default: cyclonedx-xml)
      --original-location string       specify the stored location of the original SBOM vulnerability scan result used to create this report
      --output-format string           specify the response's SBOM report format and file type format, options=[text, api-json] (default: text)
  -p, --path string                    path to file, required
      --report-uid string              specify a unique report identifier to tag this vulnerability scan result with. Supported characters: ALPHA DIGIT "-" / "." / "_" / "~"
```

## <a id='see-also'></a>See also

* [tanzu insight source](tanzu_insight_source.hbs.md)	 - Source commands
