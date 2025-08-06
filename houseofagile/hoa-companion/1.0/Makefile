ENV ?= dev

ifneq ($(ENV),$(filter $(ENV),dev staging prod))
$(error Env "$(ENV)" is not valid)
endif

# Include the appropriate env file (.env.dev or .env.prod)
include .env.$(ENV)
export

PROJECT_NAME=$(APP_NAME)

DOCKER_COMPOSE_CMD=$(shell if command -v docker-compose > /dev/null 2>&1; then echo "docker-compose"; elif docker compose version > /dev/null 2>&1; then echo "docker compose"; else echo "none"; fi)
ifeq ($(DOCKER_COMPOSE_CMD),none)
$(error "Neither docker-compose nor docker compose is available")
endif

# Use .env files in project root
DOCKER_ENV_FILE := .env.$(ENV)

# Detect which docker flavor to use (frankenphp vs standard)
DOCKER_FLAVOR := $(shell if [ -d "vendor/houseofagile/hoa-companion/hoa-docker-frankenphp" ]; then echo "frankenphp"; else echo "standard"; fi)

# Set compose files based on flavor
ifeq ($(DOCKER_FLAVOR),frankenphp)
    BASE_COMPOSE_FILE = vendor/houseofagile/hoa-companion/hoa-docker-frankenphp/docker-compose.yml
    ENV_COMPOSE_FILE = -f vendor/houseofagile/hoa-companion/hoa-docker-frankenphp/docker-compose.$(ENV).yml
else
    BASE_COMPOSE_FILE = vendor/houseofagile/hoa-companion/hoa-docker/docker-compose.yml
    ENV_COMPOSE_FILE = -f vendor/houseofagile/hoa-companion/hoa-docker/docker-compose.$(ENV).yml
endif

# Check for override file in current directory and parent directory
OVERRIDE_FILE := $(shell if [ -f "docker-compose.override.yml" ]; then echo "-f docker-compose.override.yml"; elif [ -f "../docker-compose.override.yml" ]; then echo "-f docker-compose.override.yml"; fi)

# Since we're running from project root, use direct paths
DOCKER_WORKDIR = 
BASE_COMPOSE_FILE_PATH = $(BASE_COMPOSE_FILE)
ENV_COMPOSE_FILE_PATH = $(ENV_COMPOSE_FILE)
DOCKER_ENV_FILE_PATH = ${DOCKER_ENV_FILE}

COMMON_DOCKER_COMMAND = $(DOCKER_WORKDIR) TAG=${TAG} ${DOCKER_COMPOSE_CMD} -f $(BASE_COMPOSE_FILE_PATH) $(ENV_COMPOSE_FILE_PATH) $(OVERRIDE_FILE) --env-file $(DOCKER_ENV_FILE_PATH) -p $(PROJECT_NAME)_$(ENV)

# Composer commands run inside the app service container (requires stack to be running)
COMPOSER_DOCKER_COMMAND = $(COMMON_DOCKER_COMMAND) exec app composer

build-no-cache:
	${COMMON_DOCKER_COMMAND} build --no-cache

build:
	${COMMON_DOCKER_COMMAND} build

start:
	${COMMON_DOCKER_COMMAND} up -d

start-recreate:
	${COMMON_DOCKER_COMMAND} up -d --force-recreate

stop:
	${COMMON_DOCKER_COMMAND} stop

restart: stop start

rebuild: stop build start

rm:
	${COMMON_DOCKER_COMMAND} rm

git-pull:
	git pull

deploy: git-pull stop build start

# Composer commands (requires stack to be running)
composer-install:
	${COMPOSER_DOCKER_COMMAND} install

composer-update:
	${COMPOSER_DOCKER_COMMAND} update

composer-require:
	${COMPOSER_DOCKER_COMMAND} require $(PACKAGE)

composer-remove:
	${COMPOSER_DOCKER_COMMAND} remove $(PACKAGE)

composer-dumpautoload:
	${COMPOSER_DOCKER_COMMAND} dump-autoload

composer-validate:
	${COMPOSER_DOCKER_COMMAND} validate

composer-outdated:
	${COMPOSER_DOCKER_COMMAND} outdated

# Generic composer command runner (usage: make composer CMD="show --tree")
composer:
	${COMPOSER_DOCKER_COMMAND} $(CMD)

# Help target for composer commands
help-composer:
	@echo "Available composer commands (requires stack to be running):"
	@echo "  composer-install      - Install dependencies"
	@echo "  composer-update       - Update dependencies"
	@echo "  composer-require      - Add package (usage: make composer-require PACKAGE=vendor/package)"
	@echo "  composer-remove       - Remove package (usage: make composer-remove PACKAGE=vendor/package)"
	@echo "  composer-dumpautoload - Regenerate autoloader"
	@echo "  composer-validate     - Validate composer.json"
	@echo "  composer-outdated     - Show outdated packages"
	@echo "  composer              - Run custom composer command (usage: make composer CMD='show --tree')"
	@echo ""
	@echo "Note: Start the stack first with 'make start' before running composer commands"

COMPOSE_VERSION := $(shell ${DOCKER_COMPOSE_CMD} --version 2>/dev/null | grep -oE "v?([0-9]+)\.([0-9]+)\.([0-9]+)" | head -1 | cut -d '.' -f 1)
CONTAINER_NAME_SUFFIX := $(if $(findstring 1,$(COMPOSE_VERSION)),*app_1,-app-1)
post-depl:
	@docker exec -itu developer $(PROJECT_NAME)_$(ENV)$(CONTAINER_NAME_SUFFIX) bash -c './hoa-scripts/_instance_config.sh'

# Project-specific targets (automatically detected based on docker flavor)
ifeq ($(DOCKER_FLAVOR),frankenphp)
    # FrankenPHP-specific targets
    include $(shell if [ -f "../Makefile.project" ]; then echo "../Makefile.project"; elif [ -f "Makefile.project" ]; then echo "Makefile.project"; else echo "/dev/null"; fi)
endif

# Generic project targets that work with both stacks
sh:
	$(DOCKER_WORKDIR) ${DOCKER_COMPOSE_CMD} -f $(BASE_COMPOSE_FILE_PATH) $(ENV_COMPOSE_FILE_PATH) $(OVERRIDE_FILE) --env-file $(DOCKER_ENV_FILE_PATH) -p $(PROJECT_NAME)_$(ENV) exec app bash

# Database operations (conditional based on stack)
ifeq ($(DOCKER_FLAVOR),frankenphp)
db-fresh:
	$(DOCKER_WORKDIR) ${DOCKER_COMPOSE_CMD} -f $(BASE_COMPOSE_FILE_PATH) $(ENV_COMPOSE_FILE_PATH) $(OVERRIDE_FILE) --env-file $(DOCKER_ENV_FILE_PATH) -p $(PROJECT_NAME)_$(ENV) exec app php bin/console doctrine:schema:drop --force --full-database || true
	$(DOCKER_WORKDIR) ${DOCKER_COMPOSE_CMD} -f $(BASE_COMPOSE_FILE_PATH) $(ENV_COMPOSE_FILE_PATH) $(OVERRIDE_FILE) --env-file $(DOCKER_ENV_FILE_PATH) -p $(PROJECT_NAME)_$(ENV) exec app php bin/console doctrine:migrations:migrate --no-interaction
	$(DOCKER_WORKDIR) ${DOCKER_COMPOSE_CMD} -f $(BASE_COMPOSE_FILE_PATH) $(ENV_COMPOSE_FILE_PATH) $(OVERRIDE_FILE) --env-file $(DOCKER_ENV_FILE_PATH) -p $(PROJECT_NAME)_$(ENV) exec app php bin/console app:create-default-users

db-refresh: db-fresh fixtures

fixtures:
	$(DOCKER_WORKDIR) ${DOCKER_COMPOSE_CMD} -f $(BASE_COMPOSE_FILE_PATH) $(ENV_COMPOSE_FILE_PATH) $(OVERRIDE_FILE) --env-file $(DOCKER_ENV_FILE_PATH) -p $(PROJECT_NAME)_$(ENV) exec app php bin/console doctrine:fixtures:load --no-interaction --append

fixtures-purge:
	$(DOCKER_WORKDIR) ${DOCKER_COMPOSE_CMD} -f $(BASE_COMPOSE_FILE_PATH) $(ENV_COMPOSE_FILE_PATH) $(OVERRIDE_FILE) --env-file $(DOCKER_ENV_FILE_PATH) -p $(PROJECT_NAME)_$(ENV) exec app php bin/console doctrine:fixtures:load --no-interaction --purge-with-truncate

test:
	$(DOCKER_WORKDIR) ${DOCKER_COMPOSE_CMD} -f $(BASE_COMPOSE_FILE_PATH) $(ENV_COMPOSE_FILE_PATH) $(OVERRIDE_FILE) --env-file $(DOCKER_ENV_FILE_PATH) -p $(PROJECT_NAME)_$(ENV) exec app php bin/phpunit --testsuite=Unit --stop-on-failure --no-coverage --testdox

test-all:
	$(DOCKER_WORKDIR) ${DOCKER_COMPOSE_CMD} -f $(BASE_COMPOSE_FILE_PATH) $(ENV_COMPOSE_FILE_PATH) $(OVERRIDE_FILE) --env-file $(DOCKER_ENV_FILE_PATH) -p $(PROJECT_NAME)_$(ENV) exec app php bin/phpunit --no-coverage --testdox

# Frontend targets (conditional based on stack)
js-install:
	$(DOCKER_WORKDIR) ${DOCKER_COMPOSE_CMD} -f $(BASE_COMPOSE_FILE_PATH) $(ENV_COMPOSE_FILE_PATH) $(OVERRIDE_FILE) --env-file $(DOCKER_ENV_FILE_PATH) -p $(PROJECT_NAME)_$(ENV) exec app yarn install --immutable

js-dev:
	$(DOCKER_WORKDIR) ${DOCKER_COMPOSE_CMD} -f $(BASE_COMPOSE_FILE_PATH) $(ENV_COMPOSE_FILE_PATH) $(OVERRIDE_FILE) --env-file $(DOCKER_ENV_FILE_PATH) -p $(PROJECT_NAME)_$(ENV) exec app yarn run dev

js-watch:
	$(DOCKER_WORKDIR) ${DOCKER_COMPOSE_CMD} -f $(BASE_COMPOSE_FILE_PATH) $(ENV_COMPOSE_FILE_PATH) $(OVERRIDE_FILE) --env-file $(DOCKER_ENV_FILE_PATH) -p $(PROJECT_NAME)_$(ENV) exec app yarn run dev --watch

js-build:
	$(DOCKER_WORKDIR) ${DOCKER_COMPOSE_CMD} -f $(BASE_COMPOSE_FILE_PATH) $(ENV_COMPOSE_FILE_PATH) $(OVERRIDE_FILE) --env-file $(DOCKER_ENV_FILE_PATH) -p $(PROJECT_NAME)_$(ENV) exec app yarn run build

js-clean:
	$(DOCKER_WORKDIR) ${DOCKER_COMPOSE_CMD} -f $(BASE_COMPOSE_FILE_PATH) $(ENV_COMPOSE_FILE_PATH) $(OVERRIDE_FILE) --env-file $(DOCKER_ENV_FILE_PATH) -p $(PROJECT_NAME)_$(ENV) exec app rm -rf node_modules public/build

# Messenger
consume-messages:
	$(DOCKER_WORKDIR) ${DOCKER_COMPOSE_CMD} -f $(BASE_COMPOSE_FILE_PATH) $(ENV_COMPOSE_FILE_PATH) $(OVERRIDE_FILE) --env-file $(DOCKER_ENV_FILE_PATH) -p $(PROJECT_NAME)_$(ENV) exec app php bin/console messenger:consume --time-limit=600 --memory-limit=128M --limit=1000 async
endif

# Convenience aliases for common targets
up: start
down: stop