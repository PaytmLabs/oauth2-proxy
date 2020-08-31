all:
	@echo "Usage:"
	@echo "  make create-cluster"
	@echo "  make deploy"

# create kind cluster with nginx-ingress as the most popular ingress controller for K8S
.PHONY: deploy
create-cluster:
	kind create cluster --name oauth2-proxy --config kind-cluster.yaml
	make setup-dns
	make setup-ingress

.PHONY: setup-ingress
setup-ingress:
	kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
	kubectl --namespace ingress-nginx rollout status --timeout 5m deployment/ingress-nginx-controller

# default Pod CIDR is 10.244.0.0/16 https://github.com/kubernetes-sigs/kind/blob/a6e8108025bc7a9440beedb8ef7714aec84fe87e/pkg/apis/config/v1alpha4/default.go#L52
# what makes cluster host IP equal to 10.244.0.1
# thus we add dex.localtest.me and oauth2-proxy.localtest.me stub hosts pointing to this IP
# NOT NEEDED IN REAL LIFE!
.PHONY: setup-dns
setup-dns:
	kubectl apply -f custom-dns.yaml
	kubectl -n kube-system rollout restart deployment/coredns
	kubectl -n kube-system rollout status --timeout 5m deployment/coredns

.PHONY: delete-cluster
delete-cluster:
	kind delete cluster --name oauth2-proxy

.PHONY: deploy
deploy: dependencies-example.yaml
	kustomize build | kubectl apply -f -
	kubectl rollout status --timeout 5m deployment/oauth2-proxy-example-oauth2-proxy-sample
	kubectl rollout status --timeout 1m deployment/oauth2-proxy-example-httpbin
	kubectl rollout status --timeout 1m deployment/oauth2-proxy-example-hello-world

.PHONY: undeploy
undeploy: dependencies-example.yaml
	kustomize build | kubectl delete -f -

# creates K8S dependencies manifest from helm chart
dependencies-example.yaml:
	helm dep update
	echo "# WARNING: This file is auto-generated! DO NOT EDIT MANUALLY!" > dependencies-example.yaml
	helm template --namespace default oauth2-proxy-example . >> dependencies-example.yaml
