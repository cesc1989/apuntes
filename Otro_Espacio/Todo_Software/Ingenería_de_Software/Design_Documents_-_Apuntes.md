# Design Documents - Apuntes
Apuntes de los artículos leídos en Omnivore.

# Design Docs at Google

Fuente: https://www.industrialempathy.com/posts/design-docs-at-google/


> These are relatively informal documents


> The design doc documents the high level implementation strategy and key design decisions with emphasis on the trade-offs that were considered during those decisions.


> As software engineers our job is not to produce code per se, but rather to solve problems.


> Unstructured text, like in the form of a design doc, may be the better tool for solving problems early in a project lifecycle


## Anatomy of a design doc
> Write them in whatever form makes the most sense for the particular project.
## **Context and scope**
> gives the reader a very rough overview of the landscape in which the new system is being built and what is actually being built.


> This isn’t a requirements doc. Keep it succinct!
## Goals and non-goals
> Note, that non-goals aren’t negated goals like “The system shouldn’t crash”, but rather things that could reasonably be goals, but are explicitly chosen not to be goals.


> given the context (facts), goals and non-goals (requirements), the design doc is the place to suggest solutions and show why a particular solution best satisfies those goals.
## APIs
> one should withstand the temptation to copy-paste formal interface or data definitions into the doc as these are often verbose, contain unnecessary detail
## Data storage
> copy-pasting complete schema definitions should be avoided.
## Code and pseudo-code
> Design docs should rarely contain code, or pseudo-code
## **Alternatives considered**
> The focus should be on the trade-offs that each respective design makes and how those trade-offs led to the decision to select the design that is the primary topic of the document.
## When *not* to write a design doc
> Additionally, prototyping itself may be part of the design doc creation. “I tried it out and it works” is one of the best arguments for choosing a design.
## **Implementation and iteration**
> it is inevitable that shortcomings, unaddressed requirements, or educated guesses that turned out to be wrong surface, and require changing the design. It is strongly recommended to update the design doc in this case.


# Technical Decision-Making and Alignment in a Remote Culture

Fuente: https://multithreaded.stitchfix.com/blog/2020/12/07/remote-decision-making/

## Problem Statement or Context
> The ADR opens by painting a succinct statement of the problem at hand. This might be as simple as a clear and concise problem statement, but could incorporate historical elements or communicate a desired future state
## History
> It’s important to reflect on historical solutions, to acknowledge their strengths and weaknesses
## What alternatives did we explore?
> This section is good for “showing your work” and addressing some of those suggestions in advance, documenting the advantages and disadvantages of those alternatives
## Example Code
> it’s helpful to provide sample code or even link to file diffs in a project repository. Pointing to such live code examples can help clarify your solution for people who learn best by reading code.

