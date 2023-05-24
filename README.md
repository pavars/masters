# Maģistra Darba demo

## Pre-requsites (Priekšnosacījumi)

Jābūt pieejam lokāli ieinstalētam vienam no Kubernetes klāsteriem, komandas ir izpildāmas no WSL, Linux vai MacOS operetājsistēmām.

* [Docker-Desktop](https://docs.docker.com/desktop/kubernetes/)
* [kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
* [microk8s](https://microk8s.io/docs/getting-started)

## Running (Palaišana)

1. Noklonējam git repozitoriju un ieejam koda mapē
```bash
git clone https://github.com/pavars/masters.git && cd masters
```

2. Ieinstalējam ArgoCD resursus
```bash
# Ieinstalējam argocd (reizēm jāpalaiž divas reizes, ja CRD nav laicīgi izveidojušies)
kubectl apply -k argocd/overlays/global

# Pieliekam Kubernetes anotāciju, lai argocd spētu izveidot CRD resursu
kubectl annotate crd prometheuses.monitoring.coreos.com argocd.argoproj.io/sync-options='Replace=true'

# Pārbaudām instalācijas statusu (visur jābūt READY 1/1 )
kubectl get po -n argocd

# Iegūstam noklusējuma paroli
kubectl get secrets argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d

# Ieslēdzam lokālo portu pārnešanu uz kubernetes vidi, lai piekļūtu vadības paneļiem un monitorētu statusu
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Piekļūstam lokālajai ArgoCD videi no interneta pārlūka izmantojot lietotāju admin https://127.0.0.1:8080
```

3. Gaidām, kad argocd nosinhronizēs monitoringa un žurnālēšanas rīkus Prometheus un Grafana, Loki sinhronizējam manuāli

```bash
# Pieliekam anotāciju prometheus resursam, lai tas nosinhronizētos
kubectl annotate crd prometheuses.monitoring.coreos.com argocd.argoproj.io/sync-options='Replace=true'

# Lai piekļūtu lokālajai Grafana instancei https://127.0.0.1:8081
kubectl port-forward svc/grafana -n monitoring 8081:80

# Iegūstam grafanas admin paroli
kubectl get secrets grafana -o jsonpath='{.data.admin-password}' | base64 -d
```

4. Ieslēdzam lokālo portu pārnešanu uz kubernetes vidi, lai piekļūtu vadības paneļiem
```bash
kubectl port-forward XXX yyy
```