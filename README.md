## Overview

PantryPilot AI is an intelligent recipe assistant that lets users manage a virtual pantry through natural conversation. Tell it what ingredients you have, and it suggests recipes, provides step-by-step cooking instructions, and helps you decide what to cook. All running locally on Kaggle's free GPU infrastructure.

## Key Features

- **Natural language ingredient management:** Add, remove, or clear pantry items conversationally ("I have chicken, garlic and lemon", "Remove the eggs").
- **Smart recipe suggestions:** Get 3–5 creative recipe ideas based on exactly what's in your pantry, with cooking time and difficulty.
- **Step-by-step cooking instructions:** Ask for any recipe and get a full breakdown: ingredients with measurements, numbered steps, chef's tips, and serving suggestions.
- **Streaming responses:** Recipe generation streams token-by-token so you see the answer build in real time, just like ChatGPT.
- **Session pantry memory:** Your ingredient list is remembered throughout the conversation; add and remove items freely.
- **Preference-aware suggestions:** Specify cuisine, dietary restrictions, or cooking time ("quick 20 min dinner", "vegan", "Italian") and the agent adapts.
- **Free public URL:** Runs entirely on Kaggle's free 2× T4 GPUs with a shareable Gradio link, no paid hosting required.

## Model Details

- **Model:** Gemma 4 E4B Instruct
- **Parameters:** 4 billion
- **Quantization:** 4-bit (bitsandbytes NF4)
- **VRAM usage:** ~3–4 GB 
- **Loaded via:** Unsloth `FastLanguageModel`
- **Inference:** HuggingFace `model.generate()`
- **Streaming:** `TextIteratorStreamer` in a background thread
- **Prompt format:** Gemma chat template (`<start_of_turn>user ... <start_of_turn>model`)

## Architecture

```
User Message (Gradio UI)
        │
        ▼
  Intent Router
  (keyword matching)
        │
   ┌────┴────────────────────────────────────────┐
   │                                             │
   ▼                                             ▼
Pantry Operations                    Recipe Generation
(instant response)                   (streaming response)
        │                                        │
        ▼                                        ▼
LangChain Tool.invoke()           stream_tokens() → Gradio yield
   • add_ingredients                   (Gemma 4 via Unsloth)
   • remove_ingredient
   • view_pantry                            │
   • clear_pantry                           ▼
        │                          Token-by-token streaming
        ▼                          back to Gradio ChatInterface
 Instant text reply
        │
        ▼
  [Fallback for ambiguous messages]
  LangChain ReAct AgentExecutor
  (LLM decides which tool to call)
```
