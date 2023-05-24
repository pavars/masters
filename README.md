# Maģistra Darba demo

## Pre-requsites (Priekšnosacījumi)

Jābūt pieejam lokāli ieinstalētam vienam no Kubernetes klāsteriem, komandas ir izpildāmas no WSL, Linux vai MacOS operetājsistēmām.

* [Docker-Desktop](https://docs.docker.com/desktop/kubernetes/) Pārbaudīts
* [kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation) Nav pārbaudīts
* [microk8s](https://microk8s.io/docs/getting-started) Nav pārbaudīts
* [minikube](https://minikube.sigs.k8s.io/docs/start/) Nav pārbaudīts
Un
* [ArgoCD CLI](https://argo-cd.readthedocs.io/en/stable/getting_started/)

## Running (Palaišana)

1. Noklonējam git repozitoriju un ieejam koda mapē
```bash
git clone https://github.com/pavars/masters.git && cd masters
```

2. Ieinstalējam ArgoCD resursus
```bash
# Ieinstalējam argocd (reizēm jāpalaiž divas reizes, ja CRD nav laicīgi izveidojušies)
kubectl apply -k argocd/overlays/global

# Pārbaudām instalācijas statusu (visur jābūt READY 1/1 )
kubectl get po -n argocd
# NAME                                                READY   STATUS    RESTARTS   AGE
# argocd-application-controller-0                     1/1     Running   0          103s
# argocd-applicationset-controller-6d6fc9c56b-g6nr8   1/1     Running   0          103s
# argocd-dex-server-87568f444-w87rx                   1/1     Running   0          103s
# argocd-notifications-controller-6f859d8d59-p4hnt    1/1     Running   0          103s
# argocd-redis-74d77964b-7slt5                        1/1     Running   0          103s
# argocd-repo-server-f6d876c67-ptc5q                  1/1     Running   0          103s
# argocd-server-7cd5b4d746-dfqpx                      1/1     Running   0          103s


# Iegūstam noklusējuma paroli
kubectl get secrets argocd-initial-admin-secret -o jsonpath='{.data.password}' -n argocd | base64 -d


# Ieslēdzam lokālo portu pārnešanu uz kubernetes vidi, lai piekļūtu vadības paneļiem un monitorētu statusu
# Piekļūstam lokālajai ArgoCD videi no interneta pārlūka izmantojot lietotāju admin https://127.0.0.1:8080
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

3. Sinhronizējam monitoringa sistēmas un demo lietotni

```bash
# Ja ir ieinstalēts ArgoCD CLI, tad laižam sekojošo komandu, lai forsētu resursu sinhronizāciju
argocd login localhost:8080
# WARNING: server is not configured with TLS. Proceed (y/n)? y
# Username: admin
# Password: <parole no iepriekšējām komandām>
# 'admin:login' logged in successfully
# Context 'localhost:8080' updated


# Ja nav argocd cli, tad kube-prometheus-stack-global lietotni vajag sinhronizēt no ArgoCD vadības paneļa izvēloties opciju "Replace"
argocd app sync main-app
argocd app sync loki-global
argocd app sync kube-prometheus-stack-global --replace --resource apiextensions.k8s.io:CustomResourceDefinition:prometheuses.monitoring.coreos.com
# Ja šeit iegūstam Erroru, tad apturam to no komandrindas:
# FATA[0000] rpc error: code = FailedPrecondition desc = another operation is already in progress
argocd app terminate-op kube-prometheus-stack-global

# Pieliekam anotāciju prometheus resursam, lai tas turpinātu sinhronizēties
kubectl annotate crd prometheuses.monitoring.coreos.com argocd.argoproj.io/sync-options='Replace=true'

# Lai piekļūtu lokālajai Grafana instancei https://127.0.0.1:8081
kubectl port-forward svc/grafana -n monitoring 8081:80

# Iegūstam grafanas admin paroli
kubectl get secrets grafana -n monitoring -o jsonpath='{.data.admin-password}' | base64 -d
```

4. Ieslēdzam lokālo portu pārnešanu uz kubernetes vidi, lai piekļūtu vadības paneļiem
```bash
# Grafana
kubectl port-forward svc/grafana -n monitoring 8081:80

# Prometheus
kubectl port-forward svc/kube-prometheus-stack-prometheus -n monitoring 8082:9090

# Demo (Podinfo lietotne)
kubectl port-forward svc/demo-podinfo -n demo 8082:9898


```

## Produkcijas vidē

[Produkcijas vidē](cluster-state/production/) ir jāizmanto kāds no ingress kontrolieriem (Nginx-ingress, Traefik, Istio, Linkerd, u.c. ), kas kalpo kā reverse proxy serveris un web serveris, kas servē TLS sertifikātus, domēna vārdus, sadala ienākošu datu plūsmu atbilstošajiem servisiem, nodrošina autorizāciju. Drošības konfigurāciju web serveriem jāskatās izvēlētā rīka dokumentācijā un jānokonfigurē atbilstoši prasībām. Piemēros ir izmantots Nginx ingresa kontrolieris. Konfigurācija kalpo tikai kā piemērs.
