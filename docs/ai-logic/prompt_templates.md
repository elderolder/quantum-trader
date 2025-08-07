# Prompt Templates

This document specifies the exact structure and content of the prompts that the Quantum Trader Pro application will send to external Large Language Models (LLMs) for tasks like strategy generation and market analysis.

**Audience:** Gemini (Implementation), You (Strategy & Design)
**Goal:** To create a standard, reusable set of prompts that ensures consistent and high-quality responses from the LLMs.

---

### 1. Strategy Generation Prompt

This template is used to ask an LLM to generate a trading strategy in a specific format (e.g., PineScript for TradingView).

*   **Target Model:** Mixtral 8x7B (via Groq or another provider)
*   **Purpose:** To convert a natural language request into a functional, testable trading algorithm.

#### Template Structure

The prompt will be constructed in the Python backend using an f-string or a template engine. It will combine a fixed system message with dynamic user input.

```python
def create_strategy_generation_prompt(market_conditions, user_request):
    """
    Creates the full prompt for the LLM to generate a trading strategy.
    """
    
    SYSTEM_MESSAGE = """
You are an expert quantitative trading strategist specializing in cryptocurrency markets.
Your task is to generate a complete, valid PineScript v5 trading strategy based on the user's request and the provided market conditions.

The generated script MUST include:
1. A `strategy()` declaration with a unique name.
2. Clear input controls for key parameters (e.g., moving average lengths, RSI levels).
3. Explicit entry conditions for long (`strategy.entry`) and short (`strategy.exit`) positions.
4. A basic stop-loss and take-profit mechanism.
5. Plotting of key indicators on the chart.

Do NOT include any explanatory text before or after the code block. Only output the pure PineScript code.
"""

    prompt = f"""
{SYSTEM_MESSAGE}

---
**Current Market Conditions:**
{market_conditions}

**User Strategy Request:**
"{user_request}"
---

Generate the PineScript v5 code now.
"""
    return prompt
```

Example Usage

market_conditions: "BTC/USDT is in a sideways, range-bound market between $60,000 and $65,000. Volatility is low."

user_request: "Create a mean-reversion strategy using Bollinger Bands and RSI."

The backend would generate the full prompt using the function above and send it to the Mixtral API.

2. Market Sentiment Analysis Prompt

This template is used to ask an LLM to analyze a piece of news or text and provide a structured sentiment score.

Target Model: Gemini Pro

Purpose: To extract structured sentiment data from unstructured text.

```python
def create_sentiment_analysis_prompt(article_text):
    """
    Creates the full prompt for the LLM to analyze sentiment.
    """

    prompt = f"""
Analyze the following financial news article. Provide your analysis ONLY in the form of a valid JSON object.

The JSON object must have these exact keys:
- "sentiment": A string, either "Bullish", "Bearish", or "Neutral".
- "confidence_score": A float between 0.0 and 1.0.
- "summary": A one-sentence summary of the key takeaway from the article.
- "key_topics": A JSON array of strings listing the main topics (e.g., ["inflation", "interest rates", "BTC ETF"]).

Do not include any text before or after the JSON object.

---
**Article Text:**
"{article_text}"
---

Generate the JSON response now.
"""
    return prompt
```
