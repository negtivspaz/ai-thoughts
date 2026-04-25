

## Background
After 3 months working/ playing with OpenClaw and Hermes, key is how to harness and orchestrate AI

## Boundary
- Different between OpenClaw (Hermes) and any UI of AI, such as ChatGPT, Perplexity, is when waving a more steps in a task, eg. How to write blog of traveling, in 3~ 4 section perspective: 
    - the target desitination, 
    - how/ when to go, 
    - where to stay, 
    - what to see and what to eat

After section created, ask image generation agent to read each section, summarize the section content, then create image generation prompt, for image, you'd have to tell, other than the content (which is defined in prompt)
    - which imageGenerationModel to use
    - which style
    - ratio scale
    - resolution
    
specifically for image generate. then generate the images, according to each section. after finished, embed the image back into each respective sections

all these above contains multiple steps and probably has to engage with multiple models, text model for text summarization and image for image generation

you probably need to understand what goal you want to achieve and if without ai, how would you do that... actually utilizing ai is pretty much exactly the same as if you do it manually without ai

## Architectural Thinking
- IT background
- Think ahead
- Pay attention to error, warning

## Language and Literature