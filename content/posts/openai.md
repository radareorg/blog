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