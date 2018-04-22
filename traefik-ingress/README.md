### traefik-ingress



Launch the demo Lab

```
$ cd traefik-ingress
$ kubectl apply -f .
```



Get your ELB endpoint

```
$ kubectl -n kube-system get svc/traefik-ingress-lb-svc -o jsonpath={.status.loadBalancer.ingress[*].hostname}
a91f881de463811e8aca806776b0716f-838154059.us-west-2.elb.amazonaws.com
```



check `/` for the greeting service

```
$ curl http://a91f881de463811e8aca806776b0716f-838154059.us-west-2.elb.as.com
[ERROR] please input your name in the request argument, e.g. /?name=pahud
$ curl http://a91f881de463811e8aca806776b0716f-838154059.us-west-2.elb.amazonaws.com/?name=pahud
Hello pahud
```



check `/caddy` and you should be able to see the `index.html` of [Caddy](https://caddyserver.com/).

```
$ curl http://a91f881de463811e8aca806776b0716f-838154059.us-west-2.elb.amazonaws.com/caddy
```



 check `/nginx` for nginx `index.html`

```
$ curl http://a91f881de463811e8aca806776b0716f-838154059.us-west-2.elb.amazonaws.com/nginx
```



