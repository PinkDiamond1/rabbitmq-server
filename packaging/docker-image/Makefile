# PROJECT_VERSION defaults to:
#   1. the version exported by rabbitmq-server-release;
#   2. the version stored in `git-revisions.txt`, if it exists;
#   3. a version based on git-describe(1), if it is a Git clone;
#   4. 0.0.0

PROJECT_VERSION := $(RABBITMQ_VERSION)

ifeq ($(PROJECT_VERSION),)
PROJECT_VERSION := $(shell \
if test -f git-revisions.txt; then \
	head -n1 git-revisions.txt | \
	awk '{print $$$(words $(PROJECT_DESCRIPTION) version);}'; \
else \
	(git describe --dirty --abbrev=7 --tags --always --first-parent \
	 2>/dev/null || echo rabbitmq_v0_0_0) | \
	sed -e 's/^rabbitmq_v//' -e 's/^v//' -e 's/_/./g' -e 's/-/+/' \
	 -e 's/-/./g'; \
fi)
endif

GENERIC_UNIX_ARCHIVE ?= $(wildcard $(PACKAGES_DIR)/rabbitmq-server-generic-unix-$(PROJECT_VERSION).tar.xz)

ifneq ($(filter-out clean,$(MAKECMDGOALS)),)
ifeq ($(GENERIC_UNIX_ARCHIVE),)
$(error Cannot find generic-unix archive; please specify GENERIC_UNIX_ARCHIVE)
endif

VERSION ?= $(patsubst rabbitmq-server-generic-unix-%.tar.xz,%,$(notdir $(GENERIC_UNIX_ARCHIVE)))
ifeq ($(VERSION),)
$(error Cannot determine version; please specify VERSION)
endif
endif

OTP_VERSION ?= 24.0.2
OTP_SHA256 ?= 4abca2cda7fc962ad65575e5ed834dd69c745e7e637f92cfd49f384a281d0f18
REPO ?= pivotalrabbitmq/rabbitmq
SKIP_PGP_VERIFY ?= false
PGP_KEYSERVER ?= pgpkeys.eu
ALT1_PGP_KEYSERVER ?= keyserver.ubuntu.com
ALT2_PGP_KEYSERVER ?= pgpkeys.uk

all: dist

dist:
	xzcat $(GENERIC_UNIX_ARCHIVE) | tar xvf -
	docker build --pull \
	  --build-arg SKIP_PGP_VERIFY=$(SKIP_PGP_VERIFY) \
	  --build-arg PGP_KEYSERVER=$(PGP_KEYSERVER) \
	  --build-arg OTP_VERSION=$(OTP_VERSION) \
	  --build-arg OTP_SHA256=$(OTP_SHA256) \
	  --build-arg RABBITMQ_BUILD=rabbitmq_server-$(VERSION) \
	  --tag $(REPO):$(subst +,-,$(VERSION)) \
	  .
ifdef IMAGE_TAG
	docker tag $(REPO):$(subst +,-,$(VERSION)) $(REPO):$(IMAGE_TAG)
endif

push:
	docker push $(REPO):$(subst +,-,$(VERSION))
ifdef IMAGE_TAG
	docker push $(REPO):$(IMAGE_TAG)
endif

clean:
	rm -rf rabbitmq_server-*
