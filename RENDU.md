# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom :**
**Groupe :**

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   >k3d-lab-server-0 est le master et agent-0 et agent-1 sont les deux workers. Dans docker ps, chaque nœud est un conteneur Docker rancher/k3s, avec en plus un conteneur serverlb qui sert de load balancer.
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   >Les pods qui tournent déjà sont dans le namespace kube-system. Ce sont les composants de base du cluster comme coredns, traefik, metrics-server, local-path-provisioner et svclb-traefik.

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   >Quand on supprime le pod, il disparaît et un nouveau pod avec un autre nom est recréé tout de suite.
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   >C'est le ReplicaSet (géré par le Deployment) qui surveille le cluster et relance un pod dès qu'il en manque un pour garder replicas: 1.

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   >Quand on recharge la page, le site ne marche plus. Dans /tmp/web.log, on voit que le tunnel port-forward a coupé parce qu'il était branché directement sur ce pod précis.
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   >Après avoir relancé le tunnel, mon message a disparu et la page Welcome to nginx! est revenue, car un conteneur est éphémère et repart de zéro quand il redémarre.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   >La ConfigMap garde le fichier index.html dans le cluster en dehors du conteneur et le monte dans /usr/share/nginx/html, donc la page reste même si le pod est supprimé.
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   >Les labels et sélecteurs servent à relier le Deployment et le Service aux bons pods, même quand leurs noms et leurs IP changent.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE               NOMINATED NODE   READINESS GATES
site-7b7586ddb4-9llvc   1/1     Running   0          21m   10.42.0.12   k3d-lab-server-0   <none>           <none>
site-7b7586ddb4-t4bfb   1/1     Running   0          21m   10.42.2.9    k3d-lab-agent-1    <none>           <none>
site-7b7586ddb4-xndf7   1/1     Running   0          21m   10.42.0.10   k3d-lab-server-0   <none>           <none>`.
   ```
Deux réplicas se trouvent sur `k3d-lab-server-0` et un sur `k3d-lab-agent-1`
   ```
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    >Pendant la mise à jour, les nouveaux pods nginx:1.28 démarrent et remplacent les anciens nginx:1.27 un par un, sans couper le site.
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    >Le nouveau pod bloque en ErrImagePull / ImagePullBackOff parce que l'image nginx:9.99 n'existe pas. Le site reste en ligne pendant l'erreur car les anciens pods tournent toujours, et kubectl rollout undo permet de revenir sur nginx:1.28.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    >https://fantastic-train-v649457ww7pcw9j9-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    >Dans le YAML du Deployment mario, on voit que Portainer ajoute ses propres annotations (io.portainer.kubernetes...) pour gérer l'application depuis son interface.

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    >Bonus A : Contrairement au port-forward, l'Ingress passe par Traefik et le Service, donc le site reste accessible sur le port 8081 même si on supprime un pod.
    Bonus B : Si la probe teste /nope, nginx renvoie une erreur 404 : le pod reste Running mais passe en 0/1 Ready et ne reçoit plus de trafic.
    Bonus C : drain déplace les pods de agent-0 vers les autres nœuds, mais ils ne reviennent pas tout seuls après uncordon parce que Kubernetes ne déplace pas un pod qui tourne déjà bien.
    Bonus D : Déployer dans le namespace prod crée une copie du site totalement séparée de celle dans default, ce qu'on voit bien dans Portainer.

---

**Score obtenu avec `./scripts/check.sh` :** ... / ...
