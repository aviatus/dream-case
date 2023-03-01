.PHONY: build
build:
	mkdir -p ./bin
	env GOOS=linux GOARCH=amd64 go build -o ./bin

.PHONY: build-image
build-image:
	docker build -t aviatus/validating-webhook:latest . --no-cache

.PHONY: push-image
push-image:
	docker push aviatus/validating-webhook:latest

.PHONY: build-certification
build-certification:
	kubectl apply -f ./kubernetes/certification.yaml

.PHONY: deploy
deploy:
	kubectl apply -f ./kubernetes/webhook-deployment.yaml
	kubectl apply -f ./kubernetes/webhook-service.yaml
	kubectl apply -f ./kubernetes/validating-webhook-config.yaml


.PHONY: update
update: build build-image push-image deploy

.PHONY: cleanup
cleanup:
	kubectl delete -f ./kubernetes/webhook-deployment.yaml
	kubectl delete -f ./kubernetes/webhook-service.yaml
	kubectl delete -f ./kubernetes/validating-webhook-config.yaml
