### Imperative and Declarative

##### Imperative

this creates Pod
```bash
kubectl run nginx-1 --image=nginx:latest
```

this will not create pod , just shows pod is created
```bash
kubectl run nginx-2 --image=nginx:latest --dry-run=client
```

this will not create pod & output is given in YAML and creates pod.yaml
```bash
kubectl run nginx-3 --image=nginx:latest --dry-run=client -o yaml > pod.yaml
```

this will not create pod & output is given in json and creates pod.json
```bash
kubectl run nginx-4 --image=nginx:latest --dry-run=client -o json > pod.json
```
