SHELL := /usr/bin/env bash
GOOD_FEATURE_FILES = $(shell find ../testdata/good -name "*.feature")
BAD_FEATURE_FILES  = $(shell find ../testdata/bad -name "*.feature")

TOKENS   = $(patsubst ../testdata/%.feature,acceptance/testdata/%.feature.tokens,$(GOOD_FEATURE_FILES))
ASTS     = $(patsubst ../testdata/%.feature,acceptance/testdata/%.feature.ast.ndjson,$(GOOD_FEATURE_FILES))
PICKLES  = $(patsubst ../testdata/%.feature,acceptance/testdata/%.feature.pickles.ndjson,$(GOOD_FEATURE_FILES))
SOURCES  = $(patsubst ../testdata/%.feature,acceptance/testdata/%.feature.source.ndjson,$(GOOD_FEATURE_FILES))
ERRORS   = $(patsubst ../testdata/%.feature,acceptance/testdata/%.feature.errors.ndjson,$(BAD_FEATURE_FILES))

JAVASCRIPT_FILES = $(shell find lib -name "*.js") index.js

all: .compared
.PHONY: all

.compared: .built $(TOKENS) $(ASTS) $(PICKLES) $(ERRORS) $(SOURCES)
	touch $@

.built: lib/gherkin/parser.js lib/gherkin/gherkin-languages.json $(JAVASCRIPT_FILES) dist/gherkin.js dist/gherkin.min.js node_modules/.fetched LICENSE
	./node_modules/.bin/mocha
	touch $@

node_modules/.fetched: package.json
	npm install
	touch $@

acceptance/testdata/%.feature.tokens: ../testdata/%.feature ../testdata/%.feature.tokens .built
	mkdir -p `dirname $@`
	bin/gherkin-generate-tokens $< > $@
	diff --unified $<.tokens $@
.DELETE_ON_ERROR: acceptance/testdata/%.feature.tokens

acceptance/testdata/%.feature.ast.ndjson: ../testdata/%.feature ../testdata/%.feature.ast.ndjson .built
	mkdir -p `dirname $@`
	bin/gherkin --no-source --no-pickles $< | jq --sort-keys --compact-output "." > $@
	diff --unified <(jq "." $<.ast.ndjson) <(jq "." $@)
.DELETE_ON_ERROR: acceptance/testdata/%.feature.ast.ndjson

acceptance/testdata/%.feature.pickles.ndjson: ../testdata/%.feature ../testdata/%.feature.pickles.ndjson .built
	mkdir -p `dirname $@`
	bin/gherkin --no-source --no-ast $< | jq --sort-keys --compact-output "." > $@
	diff --unified <(jq "." $<.pickles.ndjson) <(jq "." $@)
.DELETE_ON_ERROR: acceptance/testdata/%.feature.pickles.ndjson

acceptance/testdata/%.feature.source.ndjson: ../testdata/%.feature ../testdata/%.feature.source.ndjson .built
	mkdir -p `dirname $@`
	bin/gherkin --no-ast --no-pickles $< | jq --sort-keys --compact-output "." > $@
	diff --unified <(jq "." $<.source.ndjson) <(jq "." $@)
.DELETE_ON_ERROR: acceptance/testdata/%.feature.source.ndjson

acceptance/testdata/%.feature.errors.ndjson: ../testdata/%.feature ../testdata/%.feature.errors.ndjson .built
	mkdir -p `dirname $@`
	bin/gherkin $< | jq --sort-keys --compact-output "." > $@
	diff --unified <(jq "." $<.errors.ndjson) <(jq "." $@)
.DELETE_ON_ERROR: acceptance/testdata/%.feature.errors.ndjson

lib/gherkin/parser.js: ../gherkin.berp gherkin-javascript.razor ../bin/berp.exe
	mono ../bin/berp.exe -g ../gherkin.berp -t gherkin-javascript.razor -o $@
	# Remove BOM
	tail -c +4 $@ > $@.nobom
	mv $@.nobom $@

lib/gherkin/gherkin-languages.json: ../gherkin-languages.json
	cp $^ $@

dist/gherkin.js: lib/gherkin/parser.js LICENSE node_modules/.fetched
	mkdir -p `dirname $@`
	echo '/*' > $@
	cat LICENSE >> $@
	echo '*/' >> $@
	./node_modules/.bin/browserify index.js >> $@

dist/gherkin.min.js: dist/gherkin.js node_modules/.fetched
	mkdir -p `dirname $@`
	echo '/*' > $@
	cat LICENSE >> $@
	echo '*/' >> $@
	./node_modules/.bin/uglifyjs $^ >> $@

LICENSE: ../LICENSE
	cp $< $@

clean:
	rm -rf .compared .built acceptance lib/gherkin/parser.js lib/gherkin/gherkin-languages.json dist
.PHONY: clean

update-gherkin-languages: lib/gherkin/gherkin-languages.json
.PHONY: update-gherkin-languages

update-version: package.json.tmp package.json
	diff -q $< package.json || mv $< package.json
.PHONY: update-version

package.json.tmp: package.json ../VERSION
	sed "s/\(\"version\" *: *\"\)[0-9]*\.[0-9]*\.[0-9]*\(\"\)/\1`cat ../VERSION`\2/" $< > $@
.INTERMEDIATE: package.json.tmp
