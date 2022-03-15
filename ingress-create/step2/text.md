

The Nginx Ingress Controller has been installed.

Create a new *Ingress* resource called `world` for domain name `world.universe.mine`. The domain points to the K8s Node IP via `/etc/hosts`.

The *Ingress* resource should have two routes pointing to the existing *Services*:

`http://world.universe.mine:30080/europe/`

and

`http://world.universe.mine:30080/asia/`

<br>




<br>
<details><summary>Explanation</summary>
<br>

Check the NodePort *Service* for the Nginx Ingress Controller to see the ports

```plain
k -n ingress-nginx get svc ingress-nginx-controller
```

<br>

We can reach the NodePort *Service* via the K8s Node IP:

<br>

```plain
curl http://172.30.1.2:30080
```

<br>

And because of the entry in `/etc/hosts` we can call

<br>

```plain
curl http://world.universe.mine:30080
```

</details>



<br>
<details><summary>Tip 1</summary>
<br>

The *Ingress* resources needs to be created in the same *Namespace* as the applications.
</details>




<br>
<details><summary>Tip 2</summary>
<br>

You can work with this template

<br>

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: world
  namespace: world
spec:
  ingressClassName: nginx
  rules:
  - host: "world.universe.mine"
  ...
```

</details>



<br>
<details><summary>Solution</summary>
<br>

<br>

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: world
  namespace: world
spec:
  ingressClassName: nginx
  rules:
  - host: "world.universe.mine"
    http:
      paths:
      - path: /europe
        pathType: Prefix
        backend:
          service:
            name: europe
            port:
              number: 80
      - path: /asia
        pathType: Prefix
        backend:
          service:
            name: asia
            port:
              number: 80
```

</details>
