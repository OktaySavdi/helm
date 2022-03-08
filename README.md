## Helm

## Prepare Pull Request

To add a new chart, do the following in the root directory of the repo, and open a PR:

-   Add a chart directory to  `stable/`  or  `/alpha`  and place the relevant chart files there
-   Generate the tar by doing a  `helm package stable/somechart`
-   Move the generated .tgz file to  `/charts`  by doing a  `mv somechart-0.1.0.tgz charts/`
-   Re-generate the index.yaml file  `helm repo index charts/ --url https://github.com/OktaySavdi/helm/blob/main/charts/`
-   Add  `index.yaml`, the tarball, and all files in the chart directory to PR

### cli
```
helm search hub
helm search hub mysql

helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo list
helm search repo stable
helm search repo stable/mysql

helm install stable/mysql --generate-name
helm install stable/mysql --version=1.0.7

helm uninstall mysql

ls /root/.cache/helm/repository/

helm repo update

helm ls

helm create <mychart>

helm install helm-demo-configmap ./mychart --dry-run
helm install helm-demo-configmap ./mychart --dry-run --debug
helm-demo-configmap ./mychart --dry-run --debug --set namespace=lab
helm install gitops-test /root/gitops --values /root/gitops/values.yaml --namespace=myapp-test --create-namespace --set deployments.envValue=test --dry-run

helm get manifest <NAME>
```
### toYaml 
```
limits:
  default:
    cpu: 150m
    memory: 900Mi
  defaultRequest:
    cpu: 100m
    memory: 600Mi
  max:
    cpu: 1
    memory: 2100Mi

spec:
  limits:
    - type: Container
      {{- toYaml .Values.limits | nindent 6 }}
```
### Template Function
http://masterminds.github.io/sprig/strings.html
```
deployment.yml
env:
- name: {{ upper .Values.env.name }}
  value: {{ quote .Values.env.value }}

values.yaml
env:
  name: deneme
  value: 2,5,6,7

Result
env:
- name: DENEME
  value: "2,5,6,7"
```
### Template pipeline and default values
**pipeline**
```
 pipeline: {{ .Values.projectCode | upper | quote }}
 now: {{ now | date "2006-01-02" | quote }}
```
**default value**
 ```
 contact: {{ .Values.contact | default "1-800-123-0000" | quote }}
```
**Example**
```
pipeline: {{ .Values.env.name | upper | quote }}
now: {{ now | date "01-02-2006" | quote }}
contact: {{ .Values.hayda | default "1-800-123-0000" | quote }}
```
**Result**
```
pipeline: "DENEME"
now: "03-03-2022"
contact: "1-800-123-0000"
```
### Control Flow if-else
```
{{ if eq .Values.infra.region "us-e" }}ha: true{{ end }}
 ```
 ```
{{ if eq .Values.infra.region "us-e" }}
ha: true
{{ end }}
```
```    
{{- if eq .Values.infra.region "us-e" }}
ha: true
{{- end }}
```
```
rollingUpdate:
  maxSurge: 1
  {{- if gt .Values.replicaCount 2.0}}
  maxUnavailable: 0
  {{- else }}
  maxUnavailable: 1
  {{- end }}   
```
### Define scpoe using with
```
 tags:
   machine: frontdrive
   rack: 4c
   drive: ssd
   vcard: 8g
```   
```   
   {{- with .Values.tags }}
   Machine Type: {{ .machine | default "NA" | quote }}
   Rack ID: {{ .rack | quote }}
   Storage Type: {{ .drive | upper | quote }}
   Video Card: {{ .vcard | quote }}
   {{- end }}
```
### Range
 Looping with Range
```   
LangUsed:
  - Python
  - Ruby
  - Java
  - Scala
  
LangUsed: |-
  {{- range .Values.LangUsed }}
  - {{ . | title | quote }}
  {{- end }}  
```   
```  
NetSpeed: |-
  {{- range tuple "low" "medium" "high" }}
  - {{ . }}
  {{- end }}
```
### Variables
```  
tags:
  machine: frontdrive
  rack: 4c
  drive: ssd
  vcard: 8g
   
tags: 
  {{- range $key, $value := .Values.tags }}
  {{ $key }} : {{ $value }}
  {{- end }} 
```  
```  
LangUsed: |-
  {{- range $index, $topping := .Values.LangUsed }}
  - {{ $index }} : {{ $topping }}
  {{- end }} 
```  
``` 
 {{- $relname := .Release.Name -}}
 {{- with .Values.tags }}
 Machine Type: {{ .machine | default "NA" | quote }}
 Release: {{ $relname }}
 {{- end }} 
```
### Include Content Form Same File
```  
{{- define "mychart.systemlables" }}
  labels:
    drive: ssd
    machine: frontdrive
    rack: 4c
    vcard: 8g
{{- end }}

{{- template "mychart.systemlables" }}
```  
### Include scope
```
_helpers.tpl


{{- define "mychart.systemlables" }}
  labels:
    drive: ssd
    machine: frontdrive
    rack: 4c
    vcard: 8g
{{- end }}
```
### Include Template Using Keyword Include
```
 metadata:
   name: {{ .Release.Name}}-configmap
   labels:
 {{ include "mychart.version" . | indent 4 }}
 data:
   myvalue: "Sample Config Map"
   costCode: {{ .Values.costCode }}
   pipeline: {{ .Values.projectCode | upper | quote }}
   now: {{ now | date "2006-01-02"| quote }}
   contact: {{ .Values.contact | default "1-800-123-0000" | quote }}
   {{- range $key, $value := .Values.tags }}
   {{ $key }}: {{ $value | quote }}
   {{- end }}

{{include "mychart.version" $ | indent 2 }}
```  
### Notes
```
vi templates/NOTES.txt
 
 Thank you for support {{ .Chart.Name }}.
 
 Your release is named {{ .Release.Name }}.
 
 To learn more about the release, try:
 
   $ helm status {{ .Release.Name }}
   $ helm get all {{ .Release.Name }}
   $ helm uninstall {{ .Release.Name }}
 
helm install notesdemo ./mychart
```
### Sub Charts
```
cd mychart/charts

helm create mysubchart

rm -rf mysubchart/templates/*.*
---
mychart/charts/mysubchart/templates/configmap.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-innerconfig
data:
  dbhost: {{ .Values.dbhostname }}
```
### Sub Chart Global
```
mychart/values.yaml

global:
  orgdomain: com.muthu4all


mychart/charts/mysubchart/templates/configmap.yaml
and
mychart/templates/configmap.yaml


orgdomain: {{ .Values.global.orgdomain }}
```
