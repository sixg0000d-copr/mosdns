MAKEFILE_DIR := $(dir $(abspath $(lastword $(MAKEFILE_LIST))))
SCM_ROOT := $(abspath $(MAKEFILE_DIR)/..)
# SPECS
SOURCE_URL := https://github.com/IrineSistiana/mosdns
NAME := mosdns
TAG := $(shell cat $(MAKEFILE_DIR)/TAG)
VERSION := $(patsubst v%,%,$(TAG))
RELEASE := $(shell cat $(MAKEFILE_DIR)/RELEASE)
DATE := $(shell LC_ALL=en_US.UTF-8 date +'%a %b %d %Y')
# ENVIRONMENTS
spec := $(or $(spec), $(SCM_ROOT)/$(NAME).spec)
outdir := $(or $(outdir), $(SCM_ROOT)/outdir)
# PACKAGE MANAGER INSTALL COMMAND
ifeq ($(shell id -u),0)
pm_install := dnf -y install
else
pm_install := sudo dnf -y install
endif


all: srpm

init:
	$(pm_install) "@RPM Development Tools" "tar" "git" "sed" "golang >= 1.11" "patch"

$(outdir):
	[ -d "$(outdir)" ] || mkdir -p $(outdir)

spec: $(outdir) $(spec).in
	sed \
	-e "s|@SOURCE_URL@|$(SOURCE_URL)|g" \
	-e "s|@NAME@|$(NAME)|g" \
	-e "s|@TAG@|$(TAG)|g" \
	-e "s|@VERSION@|$(VERSION)|g" \
	-e "s|@RELEASE@|$(RELEASE)|g" \
	-e "s|@DATE@|$(DATE)|g" \
	$(spec).in > $(outdir)/$(NAME).spec

sources: $(outdir) init
	cd $(outdir) && \
	curl -L $(SOURCE_URL)/archive/$(TAG)/$(NAME)-$(VERSION).tar.gz -o $(NAME)-$(VERSION).tar.gz && \
	curl https://patch-diff.githubusercontent.com/raw/IrineSistiana/mosdns/pull/114.patch -o fix-sum.patch && \
	tar -xzf $(NAME)-$(VERSION).tar.gz && \
	cd $(NAME)-$(VERSION) && \
	patch -p 1 -i ../fix-sum.patch && \
	go mod vendor && \
	tar -czf ../vendor.tar.gz vendor && \
	cd .. && \
	rm -rf $(NAME)-$(VERSION)

patches: $(outdir) init
	cp $(SCM_ROOT)/*.patch $(outdir) || :

srpm: $(outdir) spec sources
	rpmbuild \
	--define "_sourcedir $(outdir)" \
	--define "_srcrpmdir $(outdir)" \
	-bs $(outdir)/$(NAME).spec

clean:
	rm -rf $(outdir)

.PHONY: default init spec sources patches srpm clean
