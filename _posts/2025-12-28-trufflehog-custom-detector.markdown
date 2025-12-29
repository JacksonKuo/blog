---
layout: post
title: "Trufflehog - Custom Detector"
date: 2025-12-29
tags: ["trufflehog"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
How to create a custom trufflehog detector?

# Summary
Create a makefile that will auto-generate a custom detector: [https://github.com/JacksonKuo/scripts-trufflehog-custom-detector](https://github.com/JacksonKuo/scripts-trufflehog-custom-detector).

Getting the Makefile syntax to be not horrifying ended up taking a while:

```
.PHONY: usage clone update-proto generate update-default custom build

# make usage name=zzz id=9000 prefix=zzz

usage:
	if [ -z "$(name)" ] || [ -z "$(id)" ] || [ -z "$(prefix)" ]; then \
		echo "Usage: make generate name=<name> id=<id> prefix=<prefix>"; \
		exit 1; \
	fi \

clone: 
	if [ ! -d "trufflehog" ]; then \
		git clone https://github.com/trufflesecurity/trufflehog; \
	fi; \
	cd trufflehog

update-proto:
	tmp="$$(mktemp)"; \
	printf '\t$(name) = $(id);\n' \
	| sed '/JWT = 1039;/r /dev/stdin' ./trufflehog/proto/detectors.proto > "$$tmp"; \
	mv "$$tmp" ./trufflehog/proto/detectors.proto


# need docker installed
generate:
	cd trufflehog && make protos
	cd trufflehog && go run hack/generate/generate.go detector $(name)

update-default:
	lower="$$(echo "$(name)" | tr 'A-Z' 'a-z')"; \
	tmp="$$(mktemp)"; \
	printf "\t\"github.com/trufflesecurity/trufflehog/v3/pkg/detectors/$$lower\"\n" \
	| sed '/"github.com\/trufflesecurity\/trufflehog\/v3\/pkg\/detectors\/copper"/r /dev/stdin' \
		./trufflehog/pkg/engine/defaults/defaults.go > "$$tmp"; \
	printf "\t\t&$$lower.Scanner{},\n" \
	| sed '/return \[\]detectors.Detector{/r /dev/stdin' \
		"$$tmp" > "$$tmp.2"; \
	mv "$$tmp.2" ./trufflehog/pkg/engine/defaults/defaults.go

custom:
	lower="$$(echo "$(name)" | tr 'A-Z' 'a-z')"; \
	dst=./trufflehog/pkg/detectors/$$lower/$$lower.go; \
	cp detector.go "$$dst"; \
	tmp="$$(mktemp)"; \
	sed \
		-e 's/PREFIX/$(prefix)/g' \
		-e 's/NAME/$(name)/g' \
		-e 's/LOWER/'"$$lower"'/g' \
		"$$dst" > "$$tmp"; \
	mv "$$tmp" "$$dst"

build:
	cd trufflehog && go build -o truffle-hog-custom ./main.go

no-ver:
	cd trufflehog && ./trufflehog-custom filesystem ./files/ --no-verification

ver: 
	cd trufflehog && ./trufflehog-custom filesystem ./files/

all: usage clone update-proto generate update-default custom build
```

I also added a simple listener server that helps observe the trufflehog verify is working correctly.

# References
[https://github.com/trufflesecurity/trufflehog/blob/main/hack/docs/Adding_Detectors_external.md](https://github.com/trufflesecurity/trufflehog/blob/main/hack/docs/Adding_Detectors_external.md)

https://trufflesecurity.com/blog/improving-trufflehog-part-i-adding-a-new-detector[](https://trufflesecurity.com/blog/improving-trufflehog-part-i-adding-a-new-detector)