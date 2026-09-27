# Weather Agent Gemini SDK Tool-Calling Assignment

An AI agent built with the Google Gemini SDK that retrieves live weather data for multiple cities using the OpenWeather API, calls the weather tool sequentially (one city at a time), and calculates the average temperature across all locations. The averaging is handled by a dedicated Python function rather than left to the model, so the result is exact rather than estimated.

This was built as Assignment 1 for an AI/ML training course. The focus of the assignment is understanding function calling (tool use) in LLM-based agents how a model can recognize it doesn't know something, call real code to find out, and then use that result in its answer.

## Table of Contents

- [Overview](#overview)
- [How It Works](#how-it-works)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Setup & Installation](#setup--installation)
- [Running the Notebook](#running-the-notebook)
- [Example Output](#example-output)
- [Design Decisions](#design-decisions)
- [Assignment Rubric Coverage](#assignment-rubric-coverage)
- [Author](#author)

## Overview

A language model on its own has no way of knowing today's temperature in any city that information isn't part of its training data, and even if it were, weather changes hourly. This project solves that using tool calling: Gemini is given access to real Python functions and decides when it needs to call them to answer a question it can't answer from memory alone.

Given a request like:

"Get the current temperature for Kathmandu, Pokhara, and Butwal, then give me the average."

the agent will:

1. Call the weather-fetching tool once per city, in order
2. Collect the three real temperature readings from OpenWeather
3. Call a second tool to calculate the exact average
4. Summarize the results in a readable report

## How It Works

```
User request
     |
     v
Gemini agent  -- recognizes it needs live data it doesn't have
     |
     v
get_current_weather(city)   <- called once per city, in sequence
     |
     +-- OpenWeather API -> Kathmandu -> 24.5 C
     +-- OpenWeather API -> Pokhara   -> 22.1 C
     +-- OpenWeather API -> Butwal    -> 27.3 C
     |
     v
calculate_average_temperature([24.5, 22.1, 27.3])
     |
     v
Average = 24.63 C
     |
     v
Final report (written by Gemini)
```

## Tech Stack

| Component | Purpose |
|---|---|
| Google Gemini SDK (`google-genai`) | LLM agent with automatic function calling |
| OpenWeather API | Live current-weather data |
| Python `requests` | HTTP calls to OpenWeather |
| Google Colab | Development and execution environment |
| Colab Secrets | Secure API key storage, kept out of source control |

## Project Structure

```
weather-agent-assignment/
├── Assignment01_Weather_Agent.ipynb   Main notebook
├── README.md                          This file
├── requirements.txt                   Python dependencies
├── .gitignore                         Excludes secrets/env/cache files
                          
```

## Setup & Installations

**1. Get API keys**

- Gemini API key: aistudio.google.com -> Get API key -> Create API key
- OpenWeather API key: openweathermap.org/api -> sign up (free) -> My API keys

Note: a newly created OpenWeather key can take up to a couple of hours to activate. A 401 error right after signup is expected just wait and retry rather than assuming something's wrong.

**2. Get the notebook**

```
git clone https://github.com/YOUR_USERNAME/weather-agent-assignment.git
```

Or open `Assignment01_Weather_Agent.ipynb` directly in Google Colab.

**3. Configure secrets in Colab**

In the notebook's left sidebar, open the Secrets panel and add:

| Name | Value |
|---|---|
| GEMINI_API_KEY | your Gemini key |
| OPENWEATHER_API_KEY | your OpenWeather key |

Turn on notebook access for both. Keys are never written into the notebook itself.

## Running the Notebook

1. Open the notebook in Colab
2. Runtime -> Run all
3. Watch the tool-call logs print in order as each city is fetched, followed by the final report

## Example Output

```
User Request: Get the current temperature for Kathmandu, Pokhara, Butwal, then give me the average.

[TOOL CALL] Fetching weather for 'Kathmandu'...
[TOOL RESULT] Kathmandu: 24.5C, clear sky
[TOOL CALL] Fetching weather for 'Pokhara'...
[TOOL RESULT] Pokhara: 22.1C, few clouds
[TOOL CALL] Fetching weather for 'Butwal'...
[TOOL RESULT] Butwal: 27.3C, haze
[TOOL CALL] Averaging [24.5, 22.1, 27.3]
[TOOL RESULT] Average -> 24.63C

WEATHER AGENT - FINAL REPORT
------------------------------------------
Kathmandu   :   24.5C  (clear sky)
Pokhara     :   22.1C  (few clouds)
Butwal      :   27.3C  (haze)
------------------------------------------
Average Temperature: 24.63C
```

Actual values will differ depending on the real weather at the time you run it.

## Design Decisions

A few choices in this project were deliberate and worth explaining rather than just leaving in the code:

- **Automatic Function Calling (AFC)** is used instead of manually intercepting and re-sending tool calls. Since the weather tool only accepts one city per call, Gemini has no way to answer without calling it three separate times, in sequence. That satisfies the "sequential tool calls" requirement naturally, without extra orchestration code.
- **The average is computed in Python, not by the model.** LLMs can occasionally get simple arithmetic slightly wrong since they're predicting likely text rather than literally calculating. A dedicated tool using `sum()/len()` removes that risk entirely.
- **API keys are never hard-coded.** Colab Secrets keeps both keys out of the notebook file completely, which is what makes it safe to put this repository up publicly.
- **Error handling** covers invalid city names, missing or invalid API keys, and network failures, so the program reports a clear message instead of crashing with a raw traceback.

## Assignment Rubric Coverage

| Criteria | Marks | How it's satisfied |
|---|---|---|
| Gemini SDK agent implementation | 1 | `client.chats.create()` with tools registered |
| Correct OpenWeather API integration | 1 | `get_current_weather()` real API call, metric units, JSON parsing |
| Sequential tool calls for 3 locations | 1 | One tool call per city, enforced by the function's design, visible in logged output |
| Correct average temperature calculation | 1 | Separate `calculate_average_temperature()` tool using exact Python arithmetic |
| Clear output, error handling & code quality | 1 | Formatted final report, try/except around all API calls, docstrings and type hints throughout |

## Author

[BIPIN PANDEY]
AI/ML Training Leapfrog




