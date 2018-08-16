# helm-chart-sandbox
Getting Started with a Helm [Chart Template](https://github.com/helm/helm/blob/master/docs/chart_template_guide/getting_started.md)

### A First Template
Create a simple chart called `helm-chart-sandbox`
```console
$ helm create helm-chart-sandbox
Creating helm-chart-sandbox
```

Let's for the basic template let's ... _remove them all!_ the template file as we will create as we go.

```console
$ rm -rf helm-chart-sandbox/templates/*.*
```

Let's begin by creating a file called `helm-chart-sandbox/templates/configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
```

The template directive `{{ .Release.Name }}` injects the release name into the template. The `Release` object is one of the built-in objects for Helm.

With this simple template, we now have an installable chart. And we can install it like this:

```console
$ helm install helm-chart-sandbox
NAME:   fallacious-hyena
LAST DEPLOYED: Thu Aug 16 13:58:42 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                        DATA  AGE
fallacious-hyena-configmap  1     0s
```

In the output above, we can see that our ConfigMap was created. Using Helm, we can retrieve the release and see the actual template that was loaded.

```console
$ helm get manifest fallacious-hyena
---
# Source: helm-chart-sandbox/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fallacious-hyena-configmap
data:
  myvalue: "Hello World"
```

Now we can delete our release: `helm delete fallacious-hyena`.
```console
$ helm delete fallacious-hyena
release "fallacious-hyena" deleted
```

Using `--dry-run` will make it easier to test your code.

```console
$ helm install --debug --dry-run helm-chart-sandbox

[debug] Created tunnel using local port: '51914'

[debug] SERVER: "127.0.0.1:51914"

[debug] Original chart version: ""
[debug] CHART PATH: /Volumes/test/helm-chart-sandbox

NAME:   volted-chinchilla
REVISION: 1
RELEASED: Thu Aug 16 14:03:05 2018
CHART: helm-chart-sandbox-0.1.0
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
affinity: {}
image:
  pullPolicy: IfNotPresent
  repository: nginx
  tag: stable
ingress:
  annotations: {}
  enabled: false
  hosts:
  - chart-example.local
  path: /
  tls: []
nodeSelector: {}
replicaCount: 1
resources: {}
service:
  port: 80
  type: ClusterIP
tolerations: []

HOOKS:
MANIFEST:

---
# Source: helm-chart-sandbox/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: volted-chinchilla-configmap
data:
  myvalue: "Hello World"
```
Values files are plain YAML files. Let's edit helm-chart-sandbox/values.yaml and then edit our ConfigMap template.

```yaml
favorite:
  drink: coffee
  food: pizza
```
Now we would have to modify the template slightly:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink }}
  food: {{ .Values.favorite.food }}
```

Let's see how this renders.

```console
$ helm install --dry-run --debug helm-chart-sandbox
[debug] Created tunnel using local port: '52414'

[debug] SERVER: "127.0.0.1:52414"

[debug] Original chart version: ""
[debug] CHART PATH: /Volumes/test/helm-chart-sandbox

NAME:   sad-opossum
REVISION: 1
RELEASED: Thu Aug 16 15:17:28 2018
CHART: helm-chart-sandbox-0.1.0
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
favorite:
  drink: coffee
  food: pizza

HOOKS:
MANIFEST:

---
# Source: helm-chart-sandbox/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: sad-opossum-configmap
data:
  myvalue: "Hello World"
  drink: coffee
  food: pizza
```

We can easily override that by adding a --set flag in our call to helm install:

```console
$ helm install --dry-run --debug --set favorite.drink=slurm helm-chart-sandbox
[debug] Created tunnel using local port: '52503'

[debug] SERVER: "127.0.0.1:52503"

[debug] Original chart version: ""
[debug] CHART PATH: /Volumes/test/helm-chart-sandbox

NAME:   ardent-abalone
REVISION: 1
RELEASED: Thu Aug 16 15:25:03 2018
CHART: helm-chart-sandbox-0.1.0
USER-SUPPLIED VALUES:
favorite:
  drink: slurm

COMPUTED VALUES:
favorite:
  drink: slurm
  food: pizza

HOOKS:
MANIFEST:

---
# Source: helm-chart-sandbox/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ardent-abalone-configmap
data:
  myvalue: "Hello World"
  drink: slurm
  food: pizza
```