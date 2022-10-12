SHELL := /bin/bash
# .PHONY: all clean build test mount_shell shell upload check-env

DOCKER_NAME := takumaynd/focal-mujoco210
DOCKER := $(shell type -p nvidia-docker || echo docker)

default: build

build:
	docker build -t $(DOCKER_NAME) .

push:
	docker push $(DOCKER_NAME)
