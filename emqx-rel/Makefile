PROJECT = emqx-rel
PROJECT_DESCRIPTION = Release Project for EMQ X Broker

# All emqx app names. Repo name, not Erlang app name
# By default, app name is the same as repo name with dash replaced by underscore.
# Otherwise define the dependency in regular erlang.mk style:
## DEPS += emqx
## dep_emqx = git https://github.com/emqx/emqx.git emqx30
DEPS += fcm lager uuid apns gun base64url voifinity_message 
dep_voifinity_message = git https://github.com/vinuvr/voifinity_message.git
dep_base64url= git https://github.com/brianloveswords/base64url.git
dep_lager= git https://github.com/erlang-lager/lager.git
dep_gun=   git https://github.com/ninenines/gun.git
dep_uuid=  git https://github.com/okeuday/uuid.git
dep_apns=  git https://github.com/inaka/apns4erl.git
dep_fcm=   git https://github.com/softwarejoint/fcm-erlang.git


OUR_APPS = emqx emqx-retainer emqx-recon emqx-reloader emqx-dashboard emqx-management \
           emqx-auth-clientid emqx-auth-username emqx-auth-ldap emqx-auth-http \
           emqx-auth-mysql emqx-auth-pgsql emqx-auth-redis emqx-auth-mongo \
           emqx-sn emqx-coap emqx-lwm2m emqx-stomp emqx-plugin-template emqx-web-hook \
           emqx-auth-jwt emqx-statsd emqx-delayed-publish emqx-lua-hook

# Default release profiles
RELX_OUTPUT_DIR ?= _rel
REL_PROFILE ?= dev
DEPLOY ?= cloud

# Default version for all OUR_APPS
## This is either a tag or branch name for ALL dependencies
EMQX_DEPS_DEFAULT_VSN ?= v3.0.0

dash = -
uscore = _

# Make Erlang app name from repo name.
# Replace dashes with underscores
app_name = $(subst $(dash),$(uscore),$(1))

# set emqx_app_name_vsn = x.y.z to override default version
app_vsn = $(if $($(call app_name,$(1))_vsn),$($(call app_name,$(1))_vsn),$(EMQX_DEPS_DEFAULT_VSN))

DEPS += $(foreach dep,$(OUR_APPS),$(call app_name,$(dep)))

# Inject variables like
# dep_app_name = git-emqx https://github.com/emqx/app-name branch-or-tag
# for erlang.mk
$(foreach dep,$(OUR_APPS),$(eval dep_$(call app_name,$(dep)) = git-emqx https://github.com/emqx/$(dep) $(call app_vsn,$(dep))))

# Add this dependency before including erlang.mk
all:: OTP_21_OR_NEWER

# COVER = true

$(shell [ -f erlang.mk ] || curl -s -o erlang.mk https://raw.githubusercontent.com/emqx/erlmk/master/erlang.mk)

include erlang.mk

# Fail fast in case older than OTP 21
.PHONY: OTP_21_OR_NEWER
OTP_21_OR_NEWER:
	@erl -noshell -eval "R = list_to_integer(erlang:system_info(otp_release)), halt(if R >= 21 -> 0; true -> 1 end)"

# Compile options
ERLC_OPTS += +warn_export_all +warn_missing_spec +warn_untyped_record

plugins:
	@rm -rf rel
	@mkdir -p rel/conf/plugins/ rel/schema/
	@for conf in $(DEPS_DIR)/*/etc/*.conf* ; do \
		if [ "emqx.conf" = "$${conf##*/}" ] ; then \
			cp $${conf} rel/conf/ ; \
		elif [ "acl.conf" = "$${conf##*/}" ] ; then \
			cp $${conf} rel/conf/ ; \
		elif [ "ssl_dist.conf" = "$${conf##*/}" ] ; then \
			cp $${conf} rel/conf/ ; \
		else \
			cp $${conf} rel/conf/plugins ; \
		fi ; \
	done
	@for schema in $(DEPS_DIR)/*/priv/*.schema ; do \
		cp $${schema} rel/schema/ ; \
	done

vm_args:
	@if [ $(DEPLOY) = "cloud" ] ; then \
		cp deps/emqx/etc/vm.args rel/conf/vm.args ; \
	else \
		cp deps/emqx/etc/vm.args.$(DEPLOY) rel/conf/vm.args ; \
	fi ;

app:: plugins vm_args vars-ln

vars-ln:
	ln -s -f vars-$(REL_PROFILE).config vars.config

