default: build
build:
	docker build -t mltools .
clean-build:
	docker build --no-cache=true -t mltools
release:
	docker tag mltools takumaynd/mltools
	docker push takumaynd/mltools
