---
layout: post
title: "Learn How to MOD Test"
author: "Mark Ashinhust"
categories: documentation
tags: [documentation,mod-testing]
image: beechcraft-denali.png
---

# Module Testing

Module Testing contains software libraries, utilities, and tools, that together form a testing framework to test a single source code module or multiple.

**MTF** stands for Module Testing Framework

## Builder
- MTF uses Boost Build ( BB ) to execute module tests. BB can build and test on Windows and Linux platforms. Executables can be build and tested, then the results will be published.
- BB manages dependencies required to produce results files.
- BB builds a dependency tree from the build files to allow for parallelization and incremental builds.

## Software Libraries

### UTF: Universal Test Framework

**C** library used to validate software modules for proper operation by exercising a modules public interface.

- A UTF library, after execution, produces a result XML file. This file is processed by MTF to apply delegate results and produce a XML or HTML file, depending on configuration.

### TST - Test Library

Necessary initialization of hardware, auxillary test libraries ( GUI Support ) and resources ( runtime libraries ) to execute a software module and its associated test.

- Provides support for initializing libraries and resources to run OpenGL mod tests with ARM sims.
