### traefic-ingress-controller





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



check `/caddy` and you should be able to see the `index.html` of Caddy.

```
$ curl http://a91f881de463811e8aca806776b0716f-838154059.us-west-2.elb.amazonaws.com/caddy
<!DOCTYPE html>
<html>
<head>
<title>Caddy</title>
<style>
    body {
        text-align: center;
        font-family: Tahoma, Geneva, Verdana, sans-serif;
    }
</style>
</head>
<body>
<h1>Caddy web server.</h1>
<p>If you see this page, Caddy container works.</p>

<p>More instructions about this image is <a href="//github.com/abiosoft/caddy-docker/blob/master/README.md" target="_blank">here</a>.<p>
<p>More instructions about Caddy is <a href="//caddyserver.com/docs" target="_blank">here</a>.<p>
</body>
</html>
```



check `/` for the greeting service

```
$ curl http://a91f881de463811e8aca806776b0716f-838154059.us-west-2.elb.as.com
[ERROR] please input your name in the request argument, e.g. /?name=pahud
$ curl http://a91f881de463811e8aca806776b0716f-838154059.us-west-2.elb.amazonaws.com/?name=pahud
Hello pahud
```



