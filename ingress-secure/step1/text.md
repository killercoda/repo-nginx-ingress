
Make sure to have solved the previous Scenario [Ingress Create](https://killercoda.com/nginx-ingress/scenario/ingress-create).

The Nginx Ingress Controller has been installed and an *Ingress* resource configured in *Namespace* `world`.

You can reach the application using 

`curl http://world.universe.mine:30080/europe`

Generate a new TLS certificate using:

`openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout cert.key -out cert.crt -subj "/CN=world.universe.mine/O=world.universe.mine"`

Configure the *Ingress* to use the new certificate, so that you can call

`curl -kv https://world.universe.mine:30443/europe`

The curl verbose output should show the new certificate being used instead of the default *Ingress* one.


<br>
<details><summary>Tip</summary>
<br>

First create a *Secret* using `k -n world create secret tls -h`

Then make the *Ingress* resource use it: `k -n world edit ing world`

</details>




<br>
<details><summary>Solution</summary>
<br>

First we generate the crt and key

<br>

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout cert.key -out cert.crt -subj "/CN=world.universe.mine/O=world.universe.mine"
```

<br>

Then we create a *Secret* from the generated files

<br>


```
kubectl -n world create secret tls ingress-tls --key cert.key --cert cert.crt
```

<br>

And finally we can make the *Ingress* use it

<br>

```
k -n world edit ing world
```

<br>

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: world
  namespace: world
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  ingressClassName: nginx
  tls:                            # add
  - hosts:                        # add
    - world.universe.mine         # add
    secretName: ingress-tls       # add
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


<br>
<details><summary>Verify</summary>
<br>

```
curl -m1 -kvI https://world.universe.mine:30443/europe 2>&1 | grep subject | grep world.universe.mine
```

</details>
