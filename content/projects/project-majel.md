+++
date = '2026-07-11T08:12:38-07:00'
draft = false
title = 'Project: Majel'
tags = ["project-majel","locall-llm","container"]
category = "project"
+++

Majel Barrett (Rodenberry) was a pillar of Star Trek, having been in the original pilot, given a new role for the series, and then going on to make appearances in both The Next Generation and Deep Space 9 while also voicing the standard Starfleet computer. It is as tribute to her I name this _Project: Majel_.

## What is Project: Majel?

Everyone who grew up watching Star Trek will remember the voice interactions with the computer, and how the computer was able to flawlessly understand what was being said and know what needed to be done, and then carry out the actions required. That is a high bar to aim for, but climbing towards it I hope to still learn a great deal, and to provide actual value to my household and the growing community of Local LLM enthusiasts.

The core of the project is a `compose` file I am using to orchestrate the various containers and services I am cobbling together to create a worthwhile local LLM stack. I will also document per service modifications and share any other pertinant information so that (hopefully) others will be able to shortcut their own local deployment.

Currently, the container stack constists of:

- Ollama: LLM server for all other services.
- Open WebUI: My primary "chat" interface and control system for Ollama.
- Searxng: A meta search service.
- Perplexica: A research tool that uses Searxng to gather information it will parse to answer specific questions posed to it.

I am also running:

- Hermes Agent: Eventually I plan to use this to make Majel accessible remotely.
- Nginx: As many services as it makes sense to do so far I am proxying access to through Nginx.
- Podman: This is my prefered container runtime, though it is 90% a drop-in-replacement for Docker.
- Ubuntu: I am running all of this on an Ubuntu LTS server that is also functioning as my daily driver (so it has a full UI, Steam, Jellyfin server, etc).
