# Nginx Ingress Controller 설치

## Helm Repository 추가

```bash
❯ helm repo add ingress-nginx <https://kubernetes.github.io/ingress-nginx>
❯ helm repo update
```

## Helm을 사용해 Nginx Ingress Controller 설치

```bash
helm install nginx-ingress ingress-nginx/ingress-nginx \
    --namespace ingress-nginx --create-namespace
```

## 설치 확인

```bash
❯ kubectl get svc -n ingress-nginx
NAME                                               TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
nginx-ingress-ingress-nginx-controller             LoadBalancer   10.111.215.81   10.0.0.240    80:32251/TCP,443:31300/TCP   3m
nginx-ingress-ingress-nginx-controller-admission   ClusterIP      10.97.125.60    <none>        443/TCP                      3m

❯ k get ingressclass
NAME    CONTROLLER             PARAMETERS   AGE
nginx   k8s.io/ingress-nginx   <none>       52m
```

## 테스트

### 테스트용 어플리케이션 배포

```bash
❯ kubectl create deployment nginx-test --image=nginx --port=80

deployment.apps/nginx-test created
❯ kubectl expose deployment nginx-test --port=80 --target-port=80 --name=nginx-test-service

service/nginx-test-service exposed
```

### 테스트용 Ingress 리소스 설정

> 기존에 사용하고 있는 kkamji.net 도메인을 사용하였습니다.
{: .prompt-info}

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-test-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: test.kkamji.net
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-test-service
            port:
              number: 80

❯ k apply -f nginx-test-ingress.yaml
ingress.networking.k8s.io/nginx-test-ingress created
```

## ingress 확인

```bash
❯ k get ingress
NAME                 CLASS   HOSTS             ADDRESS   PORTS   AGE
nginx-test-ingress   nginx   test.kkamji.net             80      4m53s
```

## test.kkamji.net으로 접속 확인

```bash
❯ curl test.kkamji.net
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="<http://nginx.org/>">nginx.org</a>.<br/>
Commercial support is available at
<a href="<http://nginx.com/>">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
