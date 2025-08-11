# Ream Validator APIs

Suppport for the validator and beacon API endpoints on the ream consensus client.

## Motivation

What problem is your project is solving? Why is it important and what area of the protocol will be affected?

Ream consenus client is a rust based implementation of the Lean Ethereunm consensus layer. Lean Ethereum is a redesign of the Beacon Ethereums consensus layer to increase the development pace of Ethereum network and make it post quantum resilinet. 

## Project description

What is your proposed solution? 
As lean chain is a redesign of beacon chain, it uses most of the current architecture which includes the existing beacon API endpoints as well. I want to help the project to implement those endpoints.

## Specification

How will you implement your solutions? Give details and more technical information on the project.
I am planning to implement the support for the `Electra` API endpoints specified by the Beacon API framework and Electra validator consensus specs.

## Roadmap

What is your proposed timeline? Outline parts of the project and insight on how much time it will take to execute them.
Implementing the necessary API endpoints which provides support to the validator client from the beacon node

## Possible challenges

What are the limitations and issues you may need to overcome?
I am still at the early stages of understanding the consensus specs and how things work in the eth clients. Lack of knowledge at the moment is the only limitation for now and also I need to brush up my skills on the rust language.

## Goal of the project

What does success look like? Describe the end goal of the project, scope, state and impact for the project to be considered finished and successful.
There are around 15 issues raised for the API endpoints. The end goal is to implement all of those.

## Collaborators

### Fellows 

Are there any fellows working with you on this project? 
I am working solo for this one.
### Mentors

Which mentors are helping you with the project? 

- Kolby 
- Ream Study group

## Resources
I am referring to the following resources 
- [eth2book](https://eth2book.info)
- EPF Study group [Youtube playlist](https://www.youtube.com/playlist?list=PLvu3JfoGPg5nt45MNYEuExw17pbH9MB3p)
