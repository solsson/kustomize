# Applying patches to a base's namespace

Scenario: there are bases for different namespaces.
An overlay needs to edit the actual namespace resource,
to add labels or annotations.
The overlay can be required to have a `namespace:` dirictive.

### Fails because the patch doesn't name a resource

```
cat <<EOF > ns.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: test
EOF

cat <<EOF > ns-annotate.yaml
apiVersion: v1
kind: Namespace
metadata:
  annotations:
    iam.amazonaws.com/permitted: .*
EOF

cat <<EOF > kustomization.yaml
namespace: test
resources:
- ns.yaml
patchesStrategicMerge:
- ns-annotate.yaml
EOF

kustomize build .
```

### Works but affects all resources

```
cat <<EOF > ns.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: test
EOF

cat <<EOF > kustomization.yaml
namespace: test
resources:
- ns.yaml
commonLabels:
  dev: "true"
EOF

kustomize build .
```

### Patching >1 namespace isn't part of the use case

If a mechanism to modify _the_ namespace is introduced,
it would conflict with such bases.
Alternatively it could modify all.

```
cat <<EOF > ns.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: test
EOF

cat <<EOF > ns2.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: test2
EOF

cat <<EOF > kustomization.yaml
namespace: test
resources:
- ns.yaml
- ns2.yaml
commonLabels:
  dev: "true"
EOF

kustomize build .
```
