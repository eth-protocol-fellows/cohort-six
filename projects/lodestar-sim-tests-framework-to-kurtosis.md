# Lodestar sim test framework migration to Kurtosis




## **Motivation**


My project aims to support the **migration of Lodestar's sim test framework** to a **Kurtosis-based framework** for test orchestration. 

Lodestar's current simulation testing framework requires manual configuration and constant maintenance for every new Ethereum fork, while most Ethereum clients are now standardised on Docker images compatible with Kurtosis.

This migration will primarily impact the testing and DevOps layers of the Lodestar protocol stack. It will enable easier and more consistent testing of upcoming Ethereum upgrades, reduces time-to-test for new features and aligns Lodestar with ecosystem-wide testing practices. 

## **Project description**


The proposed solution is to support the migration of Lodestar’s simulation testing infrastructure to Kurtosis. The approach will consist of both a **research phase** and an **implementation phase**. 

The goal is to replace the existing '**Runner**' layer, with Kurtosis and a custom minimal subset of the [ethpandaops/ethereum-package](https://github.com/ethpandaops/ethereum-package).

In details:

- Replacing the Runner with Kurtosis and ethereum-package
- Creating a minimal version of the `ethereum-package`, stripping it down to only essential services tailored to Lodestar
- **Investigate Kurtosis module development** for complex scenarios
- Understanding how to integrate Lodestar’s internal assertion framework into Kurtosis, instead of relying on external tools like Assertoor
- Adapting Lodestar assertions to Kurtosis

## **Specification**

The current Lodestar sim tests framework consists of several entities. The main components are the [Simulation](https://github.com/ChainSafe/lodestar/blob/unstable/packages/cli/test/utils/crucible/simulation.ts) orchestrator, [SimulationTracker](https://github.com/ChainSafe/lodestar/blob/unstable/packages/cli/test/utils/crucible/simulationTracker.ts), the [Runner](https://github.com/ChainSafe/lodestar/blob/unstable/packages/cli/test/utils/crucible/runner), the default assertions and [individual test scenarios](https://github.com/ChainSafe/lodestar/tree/unstable/packages/cli/test/sim) (multifork, endpoints, deneb, mixed clients, backup provider)

Research and architecture analysis:

- Identify main integration points, components adaptation versus replacement
- **ethereum-package** integration: Study its source code to understand its current configuration and conventions
- Map out the test sim [framework dependencies](https://github.com/ChainSafe/lodestar/blob/unstable/packages/cli/test/utils/crucible/simulation.ts) and understand the general orchestration workflow
- Examine how the `Runner` interacts with other components
- Document the assertions test flow
- Runner replacement: Investigate pathways to substitute Runner class with a Kurtosis-based class
- Investigate `Simulation` orchestrator and `SimulationTracker`, adaptation or replacement
- Investigate possible Kurtosis integration pathways
- Convert individual test scenarios to Kurtosis test
- CI/CD integration: Explore custom GitHub actions
- Configuration flexibility: Support both YAML and programmatic configuration
- Explore [kurtosis.yml](https://github.com/ChainSafe/lodestar/blob/unstable/.github/workflows/kurtosis.yml) future extensions and compare with [test-sim.yml](https://github.com/ChainSafe/lodestar/blob/unstable/.github/workflows/test-sim.yml)
- Fallback scenarios: Preserving old framework if Kurtosis issues arise

## **Roadmap**

The proposed timeline is intentionally adaptive to the team’s needs, with continuous feedback loops to ensure alignment. This roadmap is designed to evolve alongside team feedback and project discoveries. 

**July/end August - Exploration:**

- Deep-dive into the current sim test framework (e.g. `Runner`, `SimulationEnvironment`, `SimulationTracker`, )
- Map dependencies and execution flows
- Map possible integration points
- Identify which components can be replaced, adapted, or preserved
- Collaborate with Lodestar team to validate assumptions and align on priorities

**September/mid October - Tooling integration**

- Explore the feasibility of using the existing assertion framework inside the Kurtosis testnet
- Collaborate with team members to onboard new tests and validate the pipeline

**Mid October/November - Stabilization**:

- Final review of the integration
- Finalize documentation and transition steps
- Support Lodestar team in evaluating whether and how to migrate legacy Runner framework
- Summarize the project's outcomes and deliver a presentation at DevCon

## **Possible challenges**
While I’m still deepening my technical understanding of Ethereum’s Consensus Layer and testing infrastructure, this project offers an opportunity to learn through hands-on exploration, trial and error, and direct collaboration with the Lodestar team.

Understanding how to slim-down `ethpandaops/ethereum-package` and integrate `Kurtosis` for Lodestar's use case without creating duplication of maintenance efforts between the current sim tests framework and Kurtosis will be non-trivial.

Ensuring equivalent functionality: Effectively mapping the current simulation framework onto Kurtosis in a feature-complete way may require some architectural changes as well.

As this project is exploratory and flexible in nature, priorities may shift based on ongoing feedback from the Lodestar team mid-stream.

## **Goal of the project**

The goal of the project is to provide the Lodestar team with a workflow to easily write new test scenarios using Kurtosis. 

The scope of the project is to provide:
- A working Kurtosis'based replacement for the core sim runner
- Clear documentation and changelogs (including reasoning for changes)

A successful end state for the project would be if the team is able to:
- Run tests both locally and in CI with flexible config options
- Continue using Lodestar's internal assertion framework within Kurtosis


In order for the project to be successful, the ideal impact would be:
- Any improvement to the Lodestar test migration workflow (including associated documentation)
- The Lodestar team is more confident about its ability to adopt Kurtosis + has a cleaner and more easily maintained way to run future tests
- Add tests for new features programatically without needing to add to the internal assertion framework

Ultimately, the goal is to support the Lodestar team with a foundation they can build on, even if some parts of the legacy system remain in place for now. 



## **Collaborators**

**Fellows**

- [Irene](https://github.com/IreneBa26)

*I'm not currently collaborating with other fellows on this project, but I'm open to and would gladly consider cooperative projects. Please reach out if you're interested.*

## **Mentors**

Currently, the Lodestar team set up a dedicated thread where everyone can provide answers and clarification. Additionally, potential space has been allocated for fellows during their weekly core standups (Tuesdays at 3 PM UTC). 

In the future, one or more specific mentors may be designated for more targeted support.

## **Resources**

**Lodestar**
- [Documentation](https://chainsafe.github.io/lodestar/)
- [GitHub repository](https://github.com/ChainSafe/lodestar)
- [Simulation documentation](https://chainsafe.github.io/lodestar/contribution/testing/simulation-tests/#simulation-assertions)

**Kurtosis**
- [EPF wiki docs](https://epf.wiki/#/wiki/testing/kurtosis)
- [GitHub repository](https://github.com/kurtosis-tech/kurtosis)
- [Official documentation](https://docs.kurtosis.com/)
- [ethpandaops/ethereum-package](https://github.com/ethpandaops/ethereum-package)
- [Kurtosis deep dive guide](https://ethpandaops.io/posts/kurtosis-deep-dive/)
- [ethpandaops/assertoor](https://github.com/ethpandaops/assertoor)
- [Assertoor guide](https://ethpandaops.io/posts/assertoor-introduction/#integration-with-kurtosis)

**Blogs**
- [Ethpandaops note](https://notes.ethereum.org/@ethpandaops)
- [Ethereum Magician forum](https://ethereum-magicians.org/)
- [Ethresearch](https://ethresear.ch/)
