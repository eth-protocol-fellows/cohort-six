## Differential LLMing

TLDR: Link to the Slides introducing the project: https://docs.google.com/presentation/d/1LpxBvbz30qVwpplKAX2DC66CfZuEb7kNTB4qlZq_hHA/edit?usp=sharing 

## Motivation
Ethereum Improvement Proposals (EIPs) are implemented independently by several execution and consensus layer client teams. Each of them make their own design and implementation decisions. During development phase, this may lead to divergences which may lead to security risks, consensus breaks or edge cases that may lead to bugs. Manual peer reviews and bi-weekly All Core Dev calls are not scalable enough to tackle the above cases. Often they are found out during extensive testing phases if not later. LLMs can’t help in this case out of the box due to context length limitations. 

## Project Description
Build an AI based solution that given an EIP number and one or more client repositories, produces a human readable report that:
- Shows relevant code paths which implement several features of an EIP
- Highlight missing or non compliant logic
- Look for edge cases or security vulnerabilities
- Eventually compare two clients 

## Specification

We propose an AI based solution with the following components:
- An Orchestrator which is the brain behind the software solution. This consists of an agent (e.g. Microsoft’s Autogen) which is served using FastAPI. The agent can plan, act, summarise various features/implementations of an EIP.
- A NextJS frontend that calls the orchestrator
- Knowledgebase exposed through MCP (Model Context Protocol) servers. This consists of the following:
    - Client codebase
    - Pull requests inside the client codebase’s github repository
    - EIP documentation
    - EIP specifications and tests
    - Basic terminologies (https://github.com/prototypo/ethereum-eips-ontology) 

## Core components

### EIP Knowledgebase (KB) MCP Server
This is an MCP server that covers all the knowledgebase related to the EIP:
- ethereum/EIPs github 
- consensus-specs 
- consensus-spec-tests
- ethereum/tests.
The idea for indexing is to split markdown or code to into several chunks and split json test bundles into one row per test case. For every chunk, raw text is stored in Postgres and its corresponding embedding in a vector column.
Then we expose 3 MCP methods:
- kb_lookup(eip, section?) -> { text }
- kb_vector_search(query, k) -> [chunk_id]
- kb_test_fetch(type, id) -> YAML | JSON

### Git MCP server (one per repo)
This is for the client codebase. Similar idea as above but we also include github GraphQL API to fetch pull requests to get better ideas about implementations. 
Methods:
- git_diff(base, head) -> [chunk_id] (list only, no code)
- chunk_fetch(chunk_id) -> { code } (≤ 1 kB)
- code_search(regex) -> [chunk_id] (ripgrep-based)
- code_vector_search(query, k) -> [chunk_id] (pgvector)
- pr_list(label?) -> [ {number, merge_sha, title} ].
- pr_vector_search(k) (e.g. “show PRs similar to blob-gas cap”)

### Orchestrator
The orchestrator (based on FastAPI) is the component that:
- Exposes endpoint POST /compare { eip, clients[] }
- Initialises and discovers all MCP servers
- Governs an agent (e.g. Autogen + GPT4o) which is the core brain
The agent runs through a system prompt that works in a loop which has the following strategy:
- Plan: dividing the full work into smaller plans
- Act: e.g. MCP tool calls
- Note: description of the observation or conclusion 
The loop continues until the agent completes a full final report for a client which is a meaningful combination of the intermediate notes. 


### Frontend
Frontend is a single page based on Next js with the following minimum components:
- dropdown for EIP number
- multi-select for clients
- compare button

## Roadmap

### Month 2
- Implement core indexing and chunking logic
- Test indexing on the client code and other github repos
- Write MCP servers to expose the knowledgebase
- Test MCP servers manually to see if the responses make sense
- Write standalone Agent interface to verify if Plan-Act-Note logic works for a general case

### Month 3
- Combine the Agent with MCP servers and test
- Create FastAPI architecture for Orchestrator
- Create frontend
- Dockerise the solution and deploy in GKE or Render

### Month 4
- Get feedback from Core devs and iterate

## Possible challenges
- Choice of LLM: the best LLMs aren't free. I will need to find the best open source that I can run on my laptop for quick feedback. 
- The indexing and chunking mechanism as well as retrieval has to work well and we need to try various mechanisms for it e.g. cosine similarity. 
- The agents are a bit brittle so getting it to work well without edge cases might be a challenge.
- Getting good feedback from existing core devs is also important. 
- Prompt engineering of the agent may not necessarily go smoothly and will require lot of iterations. 

## Goal of the project

- Have a program that has found some useful edge cases or comparison or security vulnerability (if any) for existing EIP implementations. 
- Detect 2-3 known divergences across Go‑Ethereum and Erigon (or any other client combination)

## Collaborators

### Fellows 
Sato

### Mentors
Fredrik
