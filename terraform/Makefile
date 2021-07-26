BASE_DIR=123456789012/eu-central-1

all: help

prepare:
	echo 'installing brew packages'
	brew update
	brew tap liamg/tfsec
	brew install tfenv tflint terraform-docs pre-commit liamg/tfsec/tfsec coreutils checkov
	brew upgrade tfenv tflint terraform-docs pre-commit liamg/tfsec/tfsec coreutils checkov

	echo 'installing pre-commit hooks'
	pre-commit install

	echo 'setting pre-commit hooks to auto-install on clone in the future'
	git config --global init.templateDir ~/.git-template
	pre-commit init-templatedir ~/.git-template

	echo 'installing terraform with tfenv'
	tfenv install
	tfenv use

	echo 'installing infracost'
	brew install infracost
	infracost register

check-stack:
ifndef STACK
	$(error Please set STACK=[network-stack|data-stack|app-stack])
endif

init: check-stack
	cd $(STACK)/${BASE_DIR} && \
	terraform init

fmt:
	terraform fmt -recursive

validate: check-stack
	cd $(STACK)/${BASE_DIR} && \
	terraform validate

plan: check-stack
	cd $(STACK)/${BASE_DIR} && \
	terraform plan

deploy: check-stack
	cd $(STACK)/${BASE_DIR} && \
	terraform apply -auto-approve

output: check-stack
	cd $(STACK)/${BASE_DIR} && \
	terraform output

docs: check-stack
	cd $(STACK)/${BASE_DIR} && \
	terraform-docs markdown . --output-file README.md
tflint: check-stack
	cd $(STACK)/${BASE_DIR} && \
	tflint --config=../.tflint.hcl --module --deep

tfsec: check-stack
	cd $(STACK)/${BASE_DIR} && \
	tfsec .

checkov: check-stack
	cd $(STACK)/${BASE_DIR} && \
	checkov --framework terraform --output github_failed_only --soft-fail --directory .

tfcompliance: check-stack
	cd $(STACK)/${BASE_DIR} && \
	terraform plan -out=plan.out && \
	terraform-compliance --no-failure \
	-p plan.out \
	-f ../compliance
	rm -f plan.out
	rm -f plan.out.json

cost: check-stack
	cd $(STACK)/${BASE_DIR} && \
	infracost \
	--show-skipped \
	--terraform-dir . \
	--terraform-use-state

kubeconfig:
	cd app-stack/${BASE_DIR} && \
	terraform output kubectl_config
	export KUBECONFIG="${PWD}/kubeconfig_pai-data-foundation"

client_vpn:
	aws ec2 describe-client-vpn-endpoints --client-vpn-endpoint-ids

client_vpn_config:
	aws ec2 export-client-vpn-client-configuration --client-vpn-endpoint-id endpoint_id --output text>config_filename.ovpn

aws-vault:
	aws-vault exec procure-ai
help:
	@echo ">> help list:"
	@echo "  make prepare"
	@echo "  STACK=network-stack make init"
	@echo "  make fmt"
	@echo "  STACK=network-stack make validate"
	@echo "  STACK=network-stack make plan"
	@echo "  STACK=network-stack make deploy"
	@echo "  STACK=network-stack make ouput"
	@echo "  STACK=network-stack make docs"
	@echo "  STACK=network-stack make tflint"
	@echo "  STACK=network-stack make tfsec"
	@echo "  STACK=network-stack make checkov"
	@echo "  STACK=network-stack make tfcompliance"
	@echo "  STACK=network-stack make cost"
	@echo "  STACK=app-stack make kubeconfig"
