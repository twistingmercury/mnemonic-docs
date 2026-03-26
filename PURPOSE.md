# Mnemonic Docs

## PROBLEM

Mnemonic has now been split into several separate git repositories. There are some duplicate docs across the repositories, as well as missing context within the new repositories.

The repositories:

- $HOME/dev/mnemonic: (original - now will only contain the MCP tools)
- $HOME/dev/mnemonic-api: all of the RESTful endpoints used to manage patterns in mnemonic as defined in the current docs under $HOME/dev/mnemonic/docs
- $HOME/dev/mnemonic-dbs: the schema definitions and code that define the databases and build dev docker images of those databases
- $HOME/dev/mnemonic-enricher: the enrichment worker process that is responsible for all enrichment activities as defined in the current docs under $HOME/dev/mnemonic/docs
- $HOME/dev/mnemonic-admin: a web UI that consumes the apis exposed by mnemonic-api project which enables the managment of patterns.

## GOAL

The purpose of this project is to consolidate all of the documentation, architecture, design, etc., into a single project (this one) and have the other projects reference this one when design or architectural references are needed.

This project should be considered the source of truth for any documentation pertaining to the mnemonic solution going forward.
