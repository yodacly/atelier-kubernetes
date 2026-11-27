# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom :** Celayir Yelda
**Groupe :** INGE2RS1

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   > Il affiche 3 nœuds : `k3d-lab-server-0` est le master (control-plane) qui gère le cluster, et `k3d-lab-agent-0` et `k3d-lab-agent-1` sont les deux workers qui font tourner les applications.

2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   > `docker ps` montre 4 conteneurs : un par nœud du cluster (qui exécutent l'image `rancher/k3s`) plus un conteneur `k3d-lab-serverlb` qui sert de load balancer. Concrètement, dans k3d, chaque nœud est simplement un conteneur Docker qui simule une machine.

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   > La commande a créé trois objets : un `Deployment` (`web`), un `ReplicaSet` (`web-...`) et un `Pod` (`web-...`).

4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   > L'ancien pod a été supprimé et un nouveau pod avec un nom différent a été recréé tout de suite. C'est le `ReplicaSet` (géré par le `Deployment`) qui surveille le cluster et relance un pod dès qu'il en manque un pour garder `replicas: 1`.

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   > Non, le message a disparu et la page `Welcome to nginx!` est revenue, car un conteneur est éphémère et repart de l'image de base quand il est recréé.

6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   > Parce que `kubectl port-forward` ouvre un tunnel directement vers un pod précis et non vers le Service. Quand ce pod est supprimé, le tunnel coupe (`lost connection to pod` dans `/tmp/web.log`).

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   > `kubectl create` est impératif : il crée une ressource une seule fois et échoue si elle existe déjà. `kubectl apply` est déclaratif : il lit l'état voulu dans un fichier YAML, crée la ressource si elle n'existe pas, ou la met à jour si elle a été modifiée.

8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   > Le `selector` sert à repérer grâce aux labels quels pods doivent recevoir le trafic envoyé au Service. S'il ne correspond à aucun pod, le Service n'a aucun endpoint et le trafic n'aboutit nulle part (connexion refusée ou timeout).
## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   > Deux réplicas se trouvent sur `k3d-lab-server-0` et un sur `k3d-lab-agent-1` :
   > NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE               NOMINATED NODE   READINESS GATES
site-7b7586ddb4-9llvc   1/1     Running   0          21m   10.42.0.12   k3d-lab-server-0              
site-7b7586ddb4-t4bfb   1/1     Running   0          21m   10.42.2.9    k3d-lab-agent-1               
site-7b7586ddb4-xndf7   1/1     Running   0          21m   10.42.0.10   k3d-lab-server-0
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
 > Le pod avait le statut `ErrImagePull` puis `ImagePullBackOff`. C'est la commande `kubectl describe pod <nom-du-pod>` (dans la section `Events`) qui m'a donné la cause exacte : l'image `nginx:9.99` est introuvable.

11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
 > Oui, le site restait accessible parce que Kubernetes applique une mise à jour progressive (rolling update) et garde les anciens pods actifs tant que le nouveau pod n'est pas prêt (`Ready`).

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
 > [https://fantastic-train-v649457ww7pcw9j9-8585.app.github.dev/](https://fantastic-train-v649457ww7pcw9j9-8585.app.github.dev/)

13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
 > Oui, on le retrouve avec `kubectl get deploy mario`. Portainer et `kubectl` communiquent exactement avec le même composant du cluster : l'API Server de Kubernetes (situé sur le control-plane).

## Bonus (facultatif)

14. Bonus réalisés et remarques :
 > Bonus A : Contrairement au port-forward, l'Ingress passe par Traefik et le Service, donc le site reste accessible sur le port 8081 même si on supprime un pod.
 > Bonus B : Si la probe teste /nope, nginx renvoie une erreur 404 : le pod reste Running mais passe en 0/1 Ready et ne reçoit plus de trafic.
 > Bonus C : drain déplace les pods de agent-0 vers les autres nœuds, mais ils ne reviennent pas tout seuls après uncordon parce que Kubernetes ne déplace pas un pod qui tourne déjà bien.
 > Bonus D : Déployer dans le namespace prod crée une copie du site totalement séparée de celle dans default, ce qu'on voit bien dans Portainer.

---

**Score obtenu avec `./scripts/check.sh` :** 18 / 18
