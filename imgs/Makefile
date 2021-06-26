%.svg : %.dot
	dot -Tsvg -o $@ $<

.PHONY: all
all: minimal-layered-schema.svg \
  composed-minimal-layered-schema.svg \
  composed-ingested-minimal-schema.svg \
  data-model-layer.svg
