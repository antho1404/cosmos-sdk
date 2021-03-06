.PHONY: all tools tools-clean statik runsim \
protoc buf protoc-gen-buf-check-breaking protoc-gen-buf-check-lint protoc-gen-gocosmos

###
# Find OS and Go environment
# GO contains the Go binary
# FS contains the OS file separator
###
ifeq ($(OS),Windows_NT)
  GO := $(shell where go.exe 2> NUL)
  FS := "\\"
else
  GO := $(shell command -v go 2> /dev/null)
  FS := "/"
endif

ifeq ($(GO),)
  $(error could not find go. Is it in PATH? $(GO))
endif

all: tools

###############################################################################
###                                Functions                                ###
###############################################################################

go_get = $(if $(findstring Windows_NT,$(OS)),\
IF NOT EXIST $(GITHUBDIR)$(FS)$(1)$(FS) ( mkdir $(GITHUBDIR)$(FS)$(1) ) else (cd .) &\
IF NOT EXIST $(GITHUBDIR)$(FS)$(1)$(FS)$(2)$(FS) ( cd $(GITHUBDIR)$(FS)$(1) && git clone https://github.com/$(1)/$(2) ) else (cd .) &\
,\
mkdir -p $(GITHUBDIR)$(FS)$(1) &&\
(test ! -d $(GITHUBDIR)$(FS)$(1)$(FS)$(2) && cd $(GITHUBDIR)$(FS)$(1) && git clone https://github.com/$(1)/$(2)) || true &&\
)\
cd $(GITHUBDIR)$(FS)$(1)$(FS)$(2) && git fetch origin && git checkout -q $(3)

mkfile_path := $(abspath $(lastword $(MAKEFILE_LIST)))
mkfile_dir := $(shell cd $(shell dirname $(mkfile_path)); pwd)


###############################################################################
###                                 Tools                                   ###
###############################################################################

BIN ?= /usr/local/bin
UNAME_S ?= $(shell uname -s)
UNAME_M ?= $(shell uname -m)

GOPATH ?= $(shell $(GO) env GOPATH)
GITHUBDIR := $(GOPATH)$(FS)src$(FS)github.com

BUF_VERSION ?= 0.4.0

TOOLS_DESTDIR  ?= $(GOPATH)/bin
STATIK         = $(TOOLS_DESTDIR)/statik
RUNSIM         = $(TOOLS_DESTDIR)/runsim

tools: protoc buf statik runsim

# Install the runsim binary with a temporary workaround of entering an outside
# directory as the "go get" command ignores the -mod option and will polute the
# go.{mod, sum} files.
# 
# ref: https://github.com/golang/go/issues/30515
statik: $(STATIK)
$(STATIK):
	@echo "Installing statik..."
	@(cd /tmp && go get github.com/rakyll/statik@v0.1.6)

# Install the runsim binary with a temporary workaround of entering an outside
# directory as the "go get" command ignores the -mod option and will polute the
# go.{mod, sum} files.
# 
# ref: https://github.com/golang/go/issues/30515
runsim: $(RUNSIM)
$(RUNSIM):
	@echo "Installing runsim..."
	@(cd /tmp && go get github.com/cosmos/tools/cmd/runsim@v1.0.0)

PROTOC_VERSION ?= 3.11.2
ifeq ($(UNAME_S),Linux)
  PROTOC_ZIP ?= protoc-3.11.2-linux-x86_64.zip
endif
ifeq ($(UNAME_S),Darwin)
  PROTOC_ZIP ?= protoc-3.11.2-osx-x86_64.zip
endif

protoc:
	@echo "Installing protoc compiler..."
	@(cd /tmp; \
	curl -OL "https://github.com/protocolbuffers/protobuf/releases/download/v${PROTOC_VERSION}/${PROTOC_ZIP}"; \
	unzip -o ${PROTOC_ZIP} -d /usr/local bin/protoc; \
	unzip -o ${PROTOC_ZIP} -d /usr/local 'include/*'; \
	rm -f ${PROTOC_ZIP})

protoc-gen-gocosmos:
	@echo "Installing protoc-gen-gocosmos..."
	@go install github.com/regen-network/cosmos-proto/protoc-gen-gocosmos

buf: protoc-gen-buf-check-breaking protoc-gen-buf-check-lint
	@echo "Installing buf..."
	@curl -sSL \
    "https://github.com/bufbuild/buf/releases/download/v${BUF_VERSION}/buf-${UNAME_S}-${UNAME_M}" \
    -o "${BIN}/buf" && \
	chmod +x "${BIN}/buf"

protoc-gen-buf-check-breaking:
	@echo "Installing protoc-gen-buf-check-breaking..."
	@curl -sSL \
    "https://github.com/bufbuild/buf/releases/download/v${BUF_VERSION}/protoc-gen-buf-check-breaking-${UNAME_S}-${UNAME_M}" \
    -o "${BIN}/protoc-gen-buf-check-breaking" && \
	chmod +x "${BIN}/protoc-gen-buf-check-breaking"

protoc-gen-buf-check-lint:
	@echo "Installing protoc-gen-buf-check-lint..."
	@curl -sSL \
    "https://github.com/bufbuild/buf/releases/download/v${BUF_VERSION}/protoc-gen-buf-check-lint-${UNAME_S}-${UNAME_M}" \
    -o "${BIN}/protoc-gen-buf-check-lint" && \
	chmod +x "${BIN}/protoc-gen-buf-check-lint"

tools-clean:
	rm -f $(STATIK) $(GOLANGCI_LINT) $(RUNSIM)
	rm -f tools-stamp
