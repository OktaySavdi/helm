## Helm

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
