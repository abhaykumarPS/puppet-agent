NAMESPACE ?= abhay900
git_describe = $(shell git describe)
vcs_ref := $(shell git rev-parse HEAD)
build_date := $(shell date -u +%FT%T)
hadolint_available := $(shell hadolint --help > /dev/null 2>&1; echo $$?)
hadolint_command := hadolint
hadolint_container := ghcr.io/hadolint/hadolint:latest
alpine_version := 3.9
ubuntu_version := 18.04
export BUNDLE_PATH = $(PWD)/.bundle/gems
export BUNDLE_BIN = $(PWD)/.bundle/bin
export GEMFILE = $(PWD)/Gemfile
export DOCKER_BUILDKIT ?= 1

version = $(shell echo $(git_describe) | sed 's/-.*//')
dockerfile := Dockerfile
makefile_path := $(abspath $(lastword $(MAKEFILE_LIST)))
makefile_dir := $(dir $(makefile_path))
LATEST_VERSION ?= latest

prep:
	@git fetch --unshallow ||:
	@git fetch origin 'refs/tags/*:refs/tags/*'

lint:
ifeq ($(hadolint_available),0)
	@$(hadolint_command) puppet-agent-ubuntu/$(dockerfile)
	@$(hadolint_command) puppet-agent-alpine/$(dockerfile)
else
	@docker pull $(hadolint_container)
	@docker run --rm -v $(PWD)/puppet-agent-ubuntu/$(dockerfile):/Dockerfile -i $(hadolint_container) $(hadolint_command) Dockerfile
	@docker run --rm -v $(PWD)/puppet-agent-alpine/$(dockerfile):/Dockerfile -i $(hadolint_container) $(hadolint_command) Dockerfile
endif

build: 
	docker buildx build --platform="linux/amd64,linux/arm64" -t abhay900/puppet-agent:latest -f puppet-agent-ubuntu/Dockerfile --push puppet-agent-ubuntu

test: prep
	@bundle install --path $$BUNDLE_PATH --gemfile $$GEMFILE --with test
	@bundle update
	@PUPPET_TEST_DOCKER_IMAGE=$(NAMESPACE)/puppet-agent-ubuntu:$(version) \
		bundle exec --gemfile $$GEMFILE rspec spec
	@PUPPET_TEST_DOCKER_IMAGE=$(NAMESPACE)/puppet-agent-alpine:$(version) \
		bundle exec --gemfile $$GEMFILE rspec spec

publish:
	docker buildx build --platform="linux/amd64,linux/arm64" -t abhay900/puppet-agent:latest -f puppet-agent-ubuntu/Dockerfile --push .
	
.PHONY: lint build publish
