+++
date = "2023-06-02T00:00:00+00:00"
aliases = [ "openai ai" ]
draft = true 
slug = "openai-and-radare2"
title = "Applying for OpenAI's Cybersecurity grant program"
+++

## Introduction

This is a draft about the [OpenAI's recent announcement](https://openai.com/form/cybersecurity-grant-program) and how it could benefit radare2 and cybersecurity professionals at large.

There's a small pre-existing OpenAI experiment written by radare2 author, pancake, over here:

https://github.com/radareorg/radare2-extras/blob/master/openai/README.md

To say that the future features proposed are too narrow and few is an understatement. This proposal aims to change that and provide potential new contributions to the project **assisted** by GPT-4 technology, for instance, new target architectures being scaffolded by OpenAI on its different analysis levels: from decoding instructions, analysis and static emulation (via ESIL).

The following sections map what OpenAI requires on their application form and they are subject to change.

## How the funds will be used for your project

Here's a (very) rough calculation of the amount of credits required to explore and develop this idea. According to the OpenAI tokenizer:

https://platform.openai.com/tokenizer

> A helpful rule of thumb is that one token generally corresponds to ~4 characters of text for common English text. This translates to roughly ¾ of a word (so 100 tokens ~= 75 words).

This is the official pricing structure at the time of writing this proposal:

GPT-4		32K context	$0.06 / 1K tokens	$0.12 / 1K tokens
Fine-tuning	Davinci		$0.0300 / 1K tokens	$0.1200 / 1K tokens

Simplifying the pricing model and targetting the highest cost center, GPT-4 at $0.03 and $0.06/1K tokens, let's say that 1000 tokens = 1000 words, then:

radare2 % find . -iname "*.c" | xargs wc -w
(...)
    4156 ./shlr/winkd/winkd.c
     504 ./shlr/winkd/kd.c
     458 ./shlr/winkd/iob_pipe.c
       8 ./.test.c
 4265570 total

Therefore on 8K context:

>>> (4265570/1000) * 0.03
127.96710000000000000000

32K context:

>>> (4265570/1000) * 0.06
255.93420000000000000000

So rounding this up generously to not run out of credits prematurely, that's US$300 to ingest radare2's main codebase once (without headers). Assuming we might have to perform this many times it's perhaps safe to put that number at 10X, so $3000. As we get more comfortable with this work and with incremental research and fine tuning, we'll most like not reach those quotas. This is just a worst case, conservative scenario.

Then on top of that, multiple queries and responses will be made such as getting a new architecture written (which as an average of ~10000 words (find libr/anal/p/ -iname "*.c" | xargs wc -w), it'd be acceptable to top up ~$2000 in OpenAI credits.

The other $5000 will be allocated to the software engineer that implements that solution, part time, over the time span of 6 months.


## Provide a 1 year roadmap

* **2 months** for general learning and experimentation with simple examples.
* **2 months** to document and refine efficient "prompt engineering" techniques that realistically help reverse engineers perform daily tasks.
* **2 months** to devise higher complexity tasks such as generate new supported architectures.


## Set of questions or problems you hope your project will answer or address

1. Is it possible to generate a r2 target architecture decoder/disassembler/analyzer/emulator based on the other architectures present in the codebase?
2. Is it possible to assist users with firmware analysis of a microcontroller given that datasheets and other pieces of information such as CMSIS-SVD definitions have been trained on OpenAI's GPT-4 LLM?
   1. More specifically, can SVD (microcontroller peripheral mappings) be generated and applied successfully to assist firmware analysis
3. How do YARA signatures perform compared with crafted prompts against OpenAI's GPT-4?
4. Can OpenAI's code interpreter assist r2 developers and users by providing answers to general binary layout questions such as ELF sections, debug symbols, demangling, etc...
5. Explore the possibility to translate between ESIL (Radare2) and Sleight (Ghidra) architecture and static emulation languages. This is uncharted territory that can highly benefit from LLMs, extracting commonalities between the two approaches.

Those are just a small subset of questions that barely scratch the surface on how OpenAI can inform and help Radare2.


## Description of methodologies and approaches used in the project

For the aforementioned question 1, we would train a model that has specific data about the code structure of different already-implemented architectures.

On question 2, we could use https://github.com/modm-io microcontroller datasheet extraction principles to inform the LLM about the general structure and particularities of several different architectures of microcontrollers.

When exploring YARA signatures, we would start with the naive approach which would be loading the corpus of malware (and other) signatures and have GPT-4 infer some general structure of arbitrary (and/or specially crafted) input binaries.

Last but not least, we'd investigate how to extend Github Copilot-like code assistant for Radare2's codebase but using OpenAI APIs instead.

## Expected results of the project

During the first 2-4 months we expect the usefulness of our results to be relatively limited due to its experimental nature. However, as we refine our training and prompts, we could reach a point where we could significantly improve the "coverage" of supported architectures for Radare2 and accelerate its development pace.