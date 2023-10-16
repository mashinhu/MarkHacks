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

### MOCK

A small library that is needed to use the verification / validity mocking utility feature.

- Support for mocking common project facilities such as runtime assets, CPU cahce and general purpose hardware IO.

### FAKE

This is a library that provides fake implemetations of various module interfaces frequently used in an LRU.

- This usually requires a real implementation to be mocked and this mocked implementation is what is used along with the fake implementation.

## Utilities

This inclues wrapper scripts that provide homogenized functionality to build and execution environment.

### Launch Wrappers 

These are scripts that can be used by builders to launch a selected simulator. Prepares steps before executing the simulator and does clean up and / or post-processing of the output.

- The `--debug` flag option can be used for a debug simulator.
- Written in Python and has a `.py` naming convention. Located in the Boost.Build repository and tightly coupled to it.

### Delegate Wrappers

A wrapper is responsible for executing the delegate utility on UTF XML output and inserting the XML delegate result into it. This interprets the certain delegate error conditions and reports them back to the user.

### Mock Generator

This is a set of scripts as well, written in Python. These scripts use the SrcML tool to parse a source file or set of source files to produce an interface matching replacement, or with a mock implementation.

- The mock implementation can be configured to have behavior for testing.

## Tools

### Compilers
The MTF supports 

1. ARM ADSI2
2. ARM DS-5
3. MSVC
4. GCC

### Simulators
The MTF supports 

1. ARM ADS 1.2 Amulator 
2. Lauterbach Trace32 Simulator
3. ADI Visual DSP Blackfin Compiled Simulator
4. EzKit Loader

### Delegate
The MTF supports all versions of delegate.

### Results2HTML
The MTF uses the **Results2HTML** framework / script to convert UTF's XML output ( with delegate post processing ). All generate expected images included are put into a single HTML file for easy archiving into SCM.

### SrcML
The MTF uses this to parse source code.

# MOD Test Development

This section includes the best practices and how to code for MOD testing.

## Write your Tests as *Simply* as they can be

- These should be easily interpreted and code should be where we spend our focus, not on trying to decipher a test.
- The test should be similar to the requirement and makes it easier to understand. From reading the requirement and looking at the test, the reviewer should know exactly what is happening.

## Test the Case Sections

There are three main parts to a MOD test section.
1. Set up and initialization
2. Call the Function Under Test ( FUT )
3. Verification

The result will look something like this.

```c
/*------------------------------------------------------
Set up
------------------------------------------------------*/
++s_test_case_num;
UTF_sub_begin_msg( UTF_sprintf( "Test case #%d: Verify the function behavior with the "
                                "SurfaceWatch distance-remaining-callout "
                                "data values initialized to %s. "
                                "Verify the member fields are initialized to their default values.",
                   s_test_case_num,
                   test_tbl[ i ].test_str ) );
 
UTF_msg( "Reset test parameters." );
reset_test();
 
set_drc_data( &s_drc_data, &test_tbl[ i ].inp_drc_data );
 
/*------------------------------------------------------
Call FUT
------------------------------------------------------*/
UTF_msg( "Call FUT - sfw_init_drc_data()." );
sfw_init_drc_data( &s_drc_data );
 
/*------------------------------------------------------
Verify
------------------------------------------------------*/
verify_drc_data( &s_drc_data, &test_tbl[ i ].exp_drc_data, test_tbl[ i ].robust );
UTF_sub_end();
```

## Table Driven MOD Testing Practice

- Requirements can usually be exercised with multiple data points. This can be done through a table.
- A table organized inputs and reduces the MOD test code. These tables are usually recommended but not always the most efficient.
- Descriptive comments should be added to test code to show what each step in the table is doing, which helps everyone.
- You want to limit yourself to 8-10 table columns or less. This is to ease complex inputs and make everything more readable.
- Test only a few, logically related requirements in a single table. Too many requirements may make them hard to find.

*Note: Pay attention to the comments written in this code section*

```c
static const struct                 /* test table                   */
    {
    char const * const  test_str;   /* descriptive test string      */
    boolean             robust;     /* robustness test case?        */
    sfw_drc_data_type   inp_drc_data;
                                    /* input: distance-remaining-   */
                                    /* callout data values          */
    sfw_drc_data_type   exp_drc_data;
                                    /* expected: distance-remaining-*/
                                    /* callout data values          */
    }
test_tbl[] =
    { 
    { "default values", FALSE, _DRC_DATA__DFLT,                                               _DRC_DATA__DFLT },
    { "normal values",  FALSE, _DRC_DATA( 622.5f, 455.5f, SFW_DRC_2000, SFW_DRC_3000, TRUE ), _DRC_DATA__DFLT },
    },
```

## Follow the General Coding Standards

- *CodeStdCheck* code checker should work for development. The new standrads are a bit too relaxed and this older one will provide the best check for you.
- Use GVSplus in Visual Studio which has more tools and allows for better test development.
- Consider only using block comments as this makes for better formatting and alignment.
  - You can use `//` comments, but that may cause unnecessary issues and alignment errors.

## Output Print Statements

- The use of `UTF_sub_begin_msg()` and `UTF_sub_begin_nest_msg()` is encouraged for readability given the HTML format results.
- Output statements should be written in English and states what is being done and what is expected in verification.

The following are not sufficient:
1. Single word outputs.
2. Non-standard abbreviations that have not been defined.
3. Generic numerical values.

All numeric values should include units if possible.

## Use of Macros

- Macros help when table data becomes too large and overwhelming.
- Macros should be named appropriately and not at too deep of a layer. This should follow the coding conventions and guidelines.
- It is preferred to not use `stringize()`, as understanding the final result can become too difficult.
- Symbol concatenation should also be avoided with `tokcat()`. This similarly makes it hard to find and understand definitions.
- Helper functions are generally preferred over macros unless it makes sense in the situation. Visual Studio sees macros as one line so you won't be able to step through them like functions.

```c
/*------------------------------------------------------
Valid bounding boxes
------------------------------------------------------*/
{
"Test with bounding boxes near (0, 0): "\
"The input bounding box completely surrounds the expansion bounding box. "\
"The output bounding box is expected to match the input bounding box. ",
BBOX_IN _BBOX( _SCPOSN_D( 10.0, 20.0 ), _SCPOSN_D( -10.0, -20.0 ) ),
BBOX_EXPAND _BBOX( _SCPOSN_D( 5.0, 5.0 ), _SCPOSN_D( -5.0, -5.0 ) ),
BBOX_OUT _BBOX( _SCPOSN_D( 10.0, 20.0 ), _SCPOSN_D( -10.0, -20.0 ) ), FALSE
},
```

## Don't Test All Cases in one Place

- If the requirements test separate logic, they should be tested separately.
- Having un-related requirements included can create maintenance issues in the future. We need everything to be concise and in the correct place.

## Avoid Static Members of Module to be Tested

- It is not good practice to reference or use these static members at all within the MOD test.

1. This breaks scoping and information hiding in the C language. This can result in errors that would be hard to locate.
2. This also causes code coverage that can't be 100% reached.
3. This can also indicate that a member of the test is from a different level of abstraction and should be refactored into an easier test module.

## Divide up Larger Tests

- It may be difficult to manage large tests in one file. Keeping in one file may make things simpler, however, it may be better to break into multiple modules for readability.
- This makes tests easier to maintain and divide up for reviews.