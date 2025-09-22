---
title: "We don't speak JSON"
subtitle: "Why forcing LLMs to output structured data is a flawed paradigm, and what might come next for developers."
author: "David Vargas"
author_linkedin: "https://www.linkedin.com/in/dvargas92495"
reviewer: "Anita Kirkovska"
reviewer_linkedin: "https://www.linkedin.com/in/anitakirkovska"
published_date: "Sep 15, 2025"
last_updated: "Sep 15, 2025"
---

# We don't speak JSON

Why forcing LLMs to output structured data is a flawed paradigm, and what might come next for developers.

**Written by:** [David Vargas](https://www.linkedin.com/in/dvargas92495)  
**Reviewed by:** [Anita Kirkovska](https://www.linkedin.com/in/anitakirkovska)

---

Let me start with a quick story. We recently hired a new intern. Young, brilliant, fresh out of an Ivy League school, and a real go-getter. We saw a ton of potential and were excited to get them started. To ramp them up, I invited them to a customer call.

The call was gold. The customer was venting about every UX pain point and roadblock they'd hit with our product. It was a treasure trove of feedback. To make it actionable, I asked the intern, "Hey, can you prepare a spreadsheet with all the insights from that call?" I wanted a structured format so we could start chipping away at the problems.

They came back with a jumble of chaos. The columns and rows didn't make sense, and parsing the values was a nightmare. There was even a column named "Vibes," where they just wrote how they *felt* about each piece of feedback. It was a non-starter.

***Attempt two.*** "Okay," I said, "Forget the spreadsheet. Just generate a PDF report summarizing everything." A few days went by with no update. When I checked in, I saw a pretty uninspired document with very little progress. They were stuck.

Finally, I threw out the old playbook. "You know what? Just record a TikTok for me. Give me your quick reactions to the call." To my surprise, they nailed it. In two minutes, with floating text and subtitles, I got a perfectly clear, digestible summary of all the key takeaways.

This (slightly fabricated) anecdote leads to a simple realization: People perform better when they produce output in the language they were trained on.

The closer the output format is to our native way of thinking, the higher the quality of that output.

Language models are no different.

## We don't speak in JSON

So why should AI agents?

Over the last couple of years working with LLMs, I've realized they're are no different. LLMs are trained on a vast bed of natural language tokens. They perform better when they get to "speak" the language they were trained on. And I'm using "language" loosely here, I mean the fundamental format of their output.

This brings me to the main point: We, as humans, don't speak JSON. We don't communicate with each other in nested key-value pairs. JSON is a concept we constructed so our computers can perform autonomous tasks consistently.

So, I have to ask the question: why are we imposing this same limitation on our AI agents?

## How we arrived here

Let's rewind two and a half years to the early days of GPT wrappers. Our first major use case was simple: *classifying intent.* We needed to take unstructured text, like a tweet, and produce a structured output that our downstream automations and analytics could use.

![Twitter example showing a tweet that needs to be classified](images/twitter-example.avif)

### Phase 1: Asking nicely (and the Regex nightmare)

Our first attempt was just to ask the model nicely.

We'd send a prompt like, "Can you please classify the input tweet as happy, sad, or angry?" The model, being a *language* model, would often respond with a full sentence: "Sure, based on the content, I would classify this tweet as 'happy'."

![Example of asking the model nicely, showing the verbose response](images/asking-nicely-example.avif)

You could try to use regular expressions to yank out the word 'happy', but that approach gets incredibly hairy and brittle as soon as your desired output is more complex than a single word.

### Phase 2: Few-Shot prompting

As our demands increased, the community developed the technique of "one-shot" or "few-shot" prompting. We started feeding the model an example of the exact output format we wanted, forcing it to speak a language our systems could understand.

The prompt became: "...produce the output as a JSON with a top-level key called 'sentiment'. For example: `{"sentiment": "happy"}`

![Example of few-shot prompting with JSON format specification](images/few-shot-prompting-example.avif)

This worked... decently.

But it came with a host of annoying developer experience issues. How many of us have had to parse JSON out of a markdown block with triple backticks? Or dealt with a model that hallucinates a misspelled key, like `sentimant` instead of `sentiment`? We were still writing complex parsers and building logic to account for every edge case.

### Phase 3: Getting desperate (and a little threatening)

It's funny to look back on, but the next phase involved some pretty creative prompt engineering.

[Research](https://arxiv.org/abs/2307.11760) and [testing](https://www.theneuron.ai/explainer-articles/do-...) has shown that emotional prompting, like threats, can lead to better model outputs.

Our tests showed that threatening the model—"You MUST only respond with JSON. Failure to do so will have negative consequences" was improving adherence.

![Example of threatening the model to force JSON compliance](images/threatening-example.avif)

It was a clever hack, but it felt like a brute-force solution, not a first-principles approach. And it raises some... interesting questions about what we're putting in the training data for future AGI.

## The "holy crap" moment

Then, on June 13, 2023, OpenAI released Function Calling. For me, and many others, this was the "holy crap" moment. Everything clicked. This was the breakthrough that promised to let us reliably get structured output from LLMs.

By allowing us to define a schema that the model was heavily incentivized to follow, function calling felt like the key to building robust systems and automations. It was the moment we could finally start building what we now call agents. In many ways, this was **t=0 for agent development**.

## But JSON isn't the answer

Function calling has served us well, but as we build more complex agents, we're starting to see the cracks in this paradigm. The underlying problems haven't gone away.

- **No Formal Guarantees:** At the end of the day, you still can't have a 100% formal guarantee on format or schema adherence. Model providers have added layers to validate the output, but this can lead to failed requests, forcing you to build complex retry logic.

- **Probabilistic vs. Deterministic:** We are tasking a probabilistic model with a deterministic task. LLMs are famously not built for this kind of work. We're fitting a square peg into a round hole.

- **The Token Tax:** Function calling works by injecting your JSON schemas into the system prompt. A significant number of tokens are spent just defining and then generating the JSON syntax—the curly braces, the quotes, the commas, the repeated keys. These are "useless tokens" that don't contribute to the actual insight or content we want from the model.

We're doing all of this just to get the LLM to speak *our computer's* language, not its own.

## So, what's next?

This brings us to the critical question for the next generation of AI development: **Is there a way to get structured, reliable output in a format that is more language-friendly for the model?**

The answer is likely yes. We're seeing the emergence of new techniques that move beyond forcing JSON. This includes grammar-based sampling (or logit biasing) that constrains the model's output at a token level to guarantee it conforms to a specific schema, effectively building the structure as it writes.

We've explored more natural structured formats like YAML or even custom, human-readable DSLs (Domain-Specific Languages). For instance, instead of forcing the model to output a JSON diff, we can ask it for a unified diff block directly in Markdown:

```diff
- if status == "active":
- 	send_email(user)
+ if user.is_active():
+ 	notify(user)
```

This is both easy for the model to generate (it's trained heavily on code and diffs) and trivial for developers to parse. Similarly, YAML or a lightweight DSL often reduces errors while still giving downstream systems the structure they need.

The goal is to meet the model in the middle—to find a format that is both structured enough for our programs and natural enough for the LLM. Just as our intern thrived with TikTok, perhaps our agents will thrive when we stop asking for spreadsheets and start asking for something closer to their native tongue.

---

## About the Author

**David Vargas**  
*Full Stack Founding Engineer*  
[LinkedIn](https://www.linkedin.com/in/dvargas92495)

A Full-Stack Founding Engineer at Vellum, David Vargas is an MIT graduate (2017) with experience at a Series C startup and as an independent open-source engineer. He built tools for thought through his company, SamePage, and now focuses on shaping the next era of AI-driven tools for thought at Vellum.

## About the Reviewer

**Anita Kirkovska**  
*Founding Growth Lead*  
[LinkedIn](https://www.linkedin.com/in/anitakirkovska)

An AI expert with a strong ML background, specializing in GenAI and LLM education. A former Fulbright scholar, she leads Growth and Education at Vellum, helping companies build and scale AI products. She conducts LLM evaluations and writes extensively on AI best practices, empowering business leaders to drive effective AI adoption.

---

**Last Updated:** Sep 15, 2025
