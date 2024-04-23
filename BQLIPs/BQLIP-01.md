---
bqlip: 1
title: Introduce library function call 
author: Anthonyx (anthonyx@port3.io)
type: Standards Track
category: Core
status: Draft 
created: 2024-04-23
---

## Abstract

In this proposal, we explore the addition of more powerful computing capabilities to BQL, while maintaining its consistent and concise syntax.

## Motivation

In version v0.01 of the BQL script, expressions and opcodes were introduced. While this made expressions more fluent, it hindered the expansion of BQL syntax and future executor implementation. Our goal is to avoid evolving BQL into a complex language, as it would significantly raise the learning curve and complicate syntax parsing and execution.

Hence, we aim to use YAML's original structure as much as possible to describe the Workflow, facilitating easier parsing and execution. This approach ensures that users can easily comprehend BQL, and it also provides a robust syntax structure.

## Specification

During the implementation of grammar expression, replace the current expression with the semantic expression of the YAML structure. In the v0.0.1 syntax, provide corresponding methods for rewriting the expression:

```yaml
// condition if $a is little than 10, then it's true
condition: $a < 10

// calculation
slippage: $swap_amount * 0.02 + 100

// calculation
amount: floor($a)
```

The aforementioned approach necessitates the implementation of additional parsing logic. While it offers benefits in terms of semantic expression and writing, it demands complex parsing logic from the Executor. To facilitate these calculations, we should create predefined library functions and use the YAML format to define their invocation.

We will transform the aforementioned invocation method into a YAML-structured expression. Any definition that starts with an underscore indicates a system's built-in function.

```yaml
// call _lt func
condition:
  _lt: 
    - $a
    - 10

// call _mul func
amount:
  _mul:
    - $swap_amount
    - 1e18
			

// get token account for solana address
// _getAccount($mint_addr, $ADDRESS, $token_program_id)
accounts:
  userTokenAccount:
    _getAccount:
      - $mint_addr  // ctx.mint_addr
      - $ADDRESS  // user address
      - $token_program_id // contract address
```

The underscore function corresponds to a standard library function. Its sub-elements represent the parameters of the function call. During execution, the interpreter and executor identify the function and read the parameters to perform the operation. This process simplifies their work. When the parser encounters the underscore function, it directly returns the operation result to the upper-level variable as the function's outcome.

Some basic library functions:

```yaml
_gt: >
_lt: <
_gte: >=
_lte: <=
_eq: ==

_add: +
_minus: -
_mul: *
_divide: /
_mod: %

_min: Take the minimum of two numbers
_max: Take the maximum of two numbers
_floor: Round down the number
_ceil: Round up the number
_round: Round the number
```

In the implementation of the BQL Executor, follow to these principles:

1. Remove all complex expressions and implement calculations using the function call method defined in YAML.
2. Consider expressions beginning with an underscore as calls to built-in library functions.

## Rationale

During the execution of BQL, some parameters aren't fixed; they need to be calculated based on the input. However, if a language were to possess Turing-complete computing power, it would become exceptionally complex, contradicting BQL's original intention. BQL is intended to serve as a bridge between the user's intent and the specific execution, focusing on expressing the user's intent clearly without implementing overly complex logic.

By mapping the abilities that need to be expanded to a library function, we can extend BQL's capabilities and control the complexity of the language. This approach is a feasible solution.

## Backwards Compatibility

The BQL v0.0.1 version of the code still supports calculation expressions in BQL. However, versions after v0.0.1 should no longer support these expressions. There are plans to gradually phase out support for the v0.0.1 version in future releases.

Starting from version v0.0.2, a new syntax structure was introduced. This change affected the syntax of historical versions.

## Implementation

TODO

## Security Considerations

Executing the corresponding library function with underscore function doesn't significantly increase security risks. User-related signing operations are defined within the action and can only be executed after user authorization. Therefore, this procedure doesn't introduce additional security risks.
