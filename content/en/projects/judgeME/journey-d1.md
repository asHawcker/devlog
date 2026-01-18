---
date: "2026-01-13T23:07:42+05:30"
title: "Journey - JudgeME | Day 1"
toc: true
toc_position: "right"
weight: 1
---

{{< breadcrumbs >}}

---

# Journey

## Day 1

### The Initial Idea

I had this Idea from a youtube video I saw and I thought it would be fun to make and use. I also realised that this process is on the simpler side and I already had some Idea on how I would go about implementing this project.

I started with looking into what tools would be necessary and suitable for implementing the idea I had. I also wanted to follow an organised folder structure and organise the code into different files depending upon the functionality.

I did take the help of AI tools to decide on what things should be in what file and what conventions should I follow to write good readable code.

### Setting up the Github API

This was my first time using to the Oauth feature of Github. I set-up the Oauth app and then created the routes to call this API and then generating a JWT to save this info so I do not have to route requests to github to authorise everytime the LogIn button is clicked.

- _This reduces the constant need to call github and also saves up time of repeated users._

I also learned how _passport-gihub_ made this so much easy.
I proceeded with setting up the GeminiAPI and a minimal frontend to test the API.
I spend some time testing out different prompts to find what works best.

- One thing that helped was increasing the _temperature_ of the model from **1** to **1.2**. This allowed the model to be a little more "creative", which was good for the quircky response that I was expecting the model to produce.

### Frontend

I now upgraded the frontend and I decided to go with a dark neon-ish Arch linux inspired theme.
