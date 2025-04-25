# Rolling updates

- How should we update a running application?

- Strategy 1: delete old version, then deploy new version

  (not great, because it obviously provokes downtime!)

- Strategy 2: deploy new version, then delete old version

  (uses a lot of resources; also how do we shift traffic?)

- Strategy 3: replace running pods one at a time

  (sounds interesting; and good news, Kubernetes does it for us!)

---

## Rolling updates

- With rolling updates, when a Deployment is updated, it happens progressively

- The Deployment controls multiple Replica Sets

- Each Replica Set is a group of identical Pods

  (with the same image, arguments, parameters ...)

- During the rolling update, we have at least two Replica Sets:

  - the "new" set (corresponding to the "target" version)

  - at least one "old" set

- We can have multiple "old" sets

  (if we start another update before the first one is done)

---

## Update strategy

- Two parameters determine the pace of the rollout: `maxUnavailable` and `maxSurge`

- They can be specified in absolute number of pods, or percentage of the `replicas` count

- At any given time ...

  - there will always be at least `replicas`-`maxUnavailable` pods available

  - there will never be more than `replicas`+`maxSurge` pods in total

  - there will therefore be up to `maxUnavailable`+`maxSurge` pods being updated

- We have the possibility of rolling back to the previous version
  <br/>(if the update fails or is unsatisfactory in any way)

---

## Checking current rollout parameters

- Recall how we build custom reports with `kubectl` and `jq`:

.lab[

- Show the rollout plan for our deployments:
  ```bash
    kubectl get deploy -o json |
            jq ".items[] | {name:.metadata.name} + .spec.strategy.rollingUpdate"
  ```

]

---

## Rolling updates in practice

- As of Kubernetes 1.8, we can do rolling updates with:

  `deployments`, `daemonsets`, `statefulsets`

- Editing one of these resources will automatically result in a rolling update

- Rolling updates can be monitored with the `kubectl rollout` subcommand


---

## Listing versions

- We can list successive versions of a Deployment with `kubectl rollout history`

.lab[

- Look at our successive versions:
  ```bash
  kubectl rollout history deployment test
  ```

]

We don't see *all* revisions.

We might see something like 1, 4, 5.

(Depending on how many "undos" we did before.)

---

## Explaining deployment revisions

- These revisions correspond to our Replica Sets

- This information is stored in the Replica Set annotations

.lab[

- Check the annotations for our replica sets:
  ```bash
  kubectl describe replicasets -l app=test | grep -A3 ^Annotations
  ```

]

---

class: extra-details

## What about the missing revisions?

- The missing revisions are stored in another annotation:

  `deployment.kubernetes.io/revision-history`

- These are not shown in `kubectl rollout history`

- We could easily reconstruct the full list with a script

  (if we wanted to!)

---

## Rolling back to an older version

- `kubectl rollout undo` can work with a revision number

.lab[

- Roll back to the "known good" deployment version:
  ```bash
  kubectl rollout undo deployment test --to-revision=1
  ```

- Check the list of pods

]

---

class: extra-details

## Changing rollout parameters

- We want to:

  - revert to `v0.1`
  - be conservative on availability 
  - go slow on rollout speed (update only one pod at a time) 
  - give some time to our pods to "warm up" before starting more

The corresponding changes can be expressed in the following YAML snippet:

.small[
```yaml
spec:
  template:
    spec:
      containers:
      - name: test
        image: test:v0.1
  strategy:
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  minReadySeconds: 10
```
]

---

class: extra-details

## Applying changes through a YAML patch

- We could use `kubectl edit deployment test`

- But we could also use `kubectl patch` with the exact YAML shown before

.lab[

.small[

- Apply all our changes and wait for them to take effect:
  ```bash
  kubectl patch deployment test -p "
    spec:
      template:
        spec:
          containers:
          - name: test
            image: test:v0.1
      strategy:
        rollingUpdate:
          maxUnavailable: 0
          maxSurge: 1
      minReadySeconds: 10
    "
  kubectl rollout status deployment test
  kubectl get deploy -o json test |
          jq "{name:.metadata.name} + .spec.strategy.rollingUpdate"
  ```
  ] 

]

???

:EN:- Rolling updates
:EN:- Rolling back a bad deployment

:FR:- Mettre à jour un déploiement
:FR:- Concept de *rolling update* et *rollback*
:FR:- Paramétrer la vitesse de déploiement
