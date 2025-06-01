include Makefile.variables
SHELL := /bin/bash
MAKEFLAGS += --no-print-directory
ts := $(shell /bin/date "+%Y-%m-%d-%H-%M-%S")

.describe:
	@echo "Check if $(STACK_NAME) exists"
	@aws cloudformation describe-stacks \
		--stack-name $(STACK_NAME) \
		--region $(REGION) > /dev/null 2>&1;

.create:
	@echo "Stack $(STACK_NAME) does not exist, create it."
	@aws cloudformation create-stack \
		--stack-name $(STACK_NAME) \
		--template-body $(TEMPLATE_FILE) \
		--region $(REGION) \
		--capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND \
		$(if $(PARAMETERS),--parameters $(PARAMETERS)) \
		--enable-termination-protection \
		--output text && \
	 aws cloudformation wait stack-create-complete \
		--stack-name $(STACK_NAME) \
		--region $(REGION)

.update:
	@echo "Stack $(STACK_NAME) exists, update it"
	@aws cloudformation create-change-set \
		--stack-name $(STACK_NAME) \
		--template-body $(TEMPLATE_FILE) \
		--region $(REGION) \
		--capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND \
		$(if $(PARAMETERS),--parameters $(PARAMETERS)) \
		--change-set-name Update-$(ts) && \
	 aws cloudformation wait change-set-create-complete \
		--change-set-name Update-$(ts) \
		--stack-name $(STACK_NAME) \
		--region $(REGION) && \
	 aws cloudformation describe-change-set \
		--change-set-name Update-$(ts) \
		--stack-name $(STACK_NAME) \
		--region $(REGION) \
		--query "Changes[*].ResourceChange.{Action: Action, LogicalResourceId: LogicalResourceId, ResourceType: ResourceType, Replacement: Replacement, RequiresRecreation: Details[0].Target.RequiresRecreation}" \
		--output table
	@echo "Do you want to apply changes? [y/n]"
	@read confirm; \
	[[ $$confirm == y* ]] && \
		aws cloudformation execute-change-set \
			--change-set-name Update-$(ts) \
			--stack-name $(STACK_NAME) \
			--region $(REGION) || \
		aws cloudformation delete-change-set \
			--change-set-name Update-$(ts) \
			--stack-name $(STACK_NAME) \
			--region $(REGION)
		@aws cloudformation wait stack-update-complete \
			--stack-name $(STACK_NAME) \
			--region $(REGION)

.delete:
	@echo "Delete stack $(STACK_NAME) in region $(REGION)"
	@aws cloudformation update-termination-protection \
		--stack-name $(STACK_NAME) \
		--region $(REGION) \
		--no-enable-termination-protection && \
	 aws cloudformation delete-stack \
		--stack-name $(STACK_NAME) \
		--region $(REGION) && \
	 aws cloudformation wait stack-delete-complete \
		--stack-name $(STACK_NAME) \
		--region $(REGION) || \
	 echo "Stack $(STACK_NAME) deleted";

.process:
	@make .describe \
		STACK_NAME=$(STACK_NAME) \
		REGION=$(DEPLOY_REGION) 2>/dev/null && \
		(make .update \
			STACK_NAME=$(STACK_NAME) \
			REGION=$(DEPLOY_REGION) \
			PARAMETERS=$(PARAMETERS) \
			TEMPLATE_FILE=$(TEMPLATE_FILE) 2>&1 | tee /dev/tty | grep -q 'Stack update complete' \
			|| true) || \
		make .create \
			STACK_NAME=$(STACK_NAME) \
			REGION=$(DEPLOY_REGION) \
			PARAMETERS=$(PARAMETERS) \
			TEMPLATE_FILE=$(TEMPLATE_FILE)

vpc:
	@make .process \
		STACK_NAME=$(PROJECT)-vpc \
		REGION=$(DEPLOY_REGION) \
		TEMPLATE_FILE=file://week1/vpc/vpc.yaml \
		PARAMETERS=file://week1/vpc/vpc.json

instances:
	@make .process \
		STACK_NAME=$(PROJECT)-instances \
		REGION=$(DEPLOY_REGION) \
		TEMPLATE_FILE=file://week1/instances/instances.yaml \
		PARAMETERS=file://week1/instances/instances.json
