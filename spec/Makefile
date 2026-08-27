# Build the ACS specification with Bikeshed.
#
# Bikeshed needs Python 3.12+ and a one-time data download:
#
#   uv tool install --python 3.12 bikeshed
#   bikeshed update
#
# See README.md in this directory.

BIKESHED ?= bikeshed
SRC      := index.bs
OUT      := index.html
DEPS     := $(SRC) copyright.include $(wildcard sections/*.include) $(wildcard diagrams/*)

.PHONY: all lint watch clean update

# Build the spec. Warnings are errors: a broken cross-reference or a dfn
# nothing links to is a defect in the spec, not noise to scroll past.
all: $(OUT)

$(OUT): $(DEPS)
	$(BIKESHED) --die-on=warning spec $(SRC) $(OUT)

# Same check, no output file. This is what CI runs.
lint:
	$(BIKESHED) --die-on=warning spec $(SRC) /dev/null

# Rebuild on every save while editing.
watch:
	$(BIKESHED) watch $(SRC) $(OUT)

# Refresh Bikeshed's cross-reference and bibliography data.
update:
	$(BIKESHED) update

clean:
	rm -f $(OUT)
