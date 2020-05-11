# Charts
- [Charts](#charts)
  - [Basics](#basics)
  - [Built-in Objects](#built-in-objects)
  - [Value files](#value-files)
  - [Templates, Functions and Pipelines](#templates-functions-and-pipelines)
    - [Templates](#templates)
    - [Pipelines](#pipelines)
    - [Lookup function](#lookup-function)
  - [Flow control](#flow-control)
  - [Variables](#variables)


## Basics
- A chart is organized as a collection of files inside of a directory
![alt text](./chart-dir-structure.png "Logo Title Text 1")

- The **templates/** directory is for template files that helm renders through template rendering engine and send it to kubernetes
- The **values.yaml** contains value for a chart
- **Chart.yaml** contains description of the chart. **charts** directory contains other charts(**subcharts**)
- ```helm get manifest <release-name>``` takes a release name and prints out all the kubernetes resources that were uploaded to the server
  - Each file begins with **---**
  - Followed by an automatically generated comment line that tells us what template file generated this YAML document.
- ```helm list``` - lists all the release in helm

## Built-in Objects 
- Objects are passed into template from the templating engine
- Objects can have one value, a function or another object
- [Link for the preset object](https://helm.sh/docs/chart_template_guide/builtin_objects/)
- The built-in values always begin with a capital letter

## Value files
- Content of **Value** object can come from multiple sources.
  - **values.yml** file
  - **values.yml** of parent if this is a subchart
  - passed through command line - ```helm install -f myvals.yaml ./mychart```
  - ```set``` parameter - ```helm install --set foo=bar ./mychart```
- Above list is in order of specificity. 
  - **values.yaml** is the default, which can be overridden by a parent chart's **values.yaml**, which can in turn be overridden by a user-supplied values file, which can in turn be overridden by ```--set``` parameters.
- To delete default key, just set it to ```null```

## Templates, Functions and Pipelines

### Templates
- Template function has the syntax of ```functionName arg1 arg2 ...```
- Total 60 available function in [Go template language](https://godoc.org/text/template) and [Sprig template library](https://masterminds.github.io/sprig/)

### Pipelines
- We can send the argument to the function using pipe ```|```
  - ```.Values.favoriteDrink | upper | quote```

### Lookup function
- ```lookup``` function can be used to look up resources in a running cluster
- When lookup returns an object, it will return a dictionary

```go
{{ range $index, $service := (lookup "v1" "Service" "mynamespace" "").items }}
    {{/* do something with each service */}}
{{ end }}
```

## Flow control
- If/Else
```go
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
```
- The **|-** marker in YAML takes a multi-line string. This can be a useful technique for embedding big blocks of data inside of your manifests, as exemplified here.
```yaml
sizes: |-
    {{- range tuple "small" "medium" "large" }}
    - {{ . }}
    {{- end }}
```
Above will produce
```yaml
sizes: |-
  - small
  - medium
  - large
```

## Variables

- Below code will fail as `.Release.Name` is out of scope
```yaml
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ .Release.Name }}
  {{- end }}
```
- One way to workaround is to define variable 
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Release.Name}}-config
data:
  myvalue: "Hello world"
  {{- $relName = .Release.Name -}} # delete this line totally when transpiling to yaml
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ $relname }}
  {{- end }}
```

- Above will produce
```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: viable-badger-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
  release: viable-badger
```
- Variables are useful in rarnge loop
```yaml
 toppings: |-
    {{- range $index, $topping := .Values.pizzaToppings }}
      {{ $index }}: {{ $topping }}
    {{- end }}
```

- there is one variable that is always global - $ - this variable will always point to the root context