# PRD: AI Flight Finder

## 1. Introduction / Overview

AI Flight Finder is a Python-based command-line tool that lets users search for round-trip flights using plain English. Instead of filling out rigid search forms, users describe what they want (e.g., "Find me a round trip from New York to London in late June, nonstop, leaving in the morning") and the tool parses their intent, queries flight data, and returns smart, ranked recommendations.

The tool is built with **Pydantic AI** to handle agent logic, structured data modeling, and LLM-powered natural language understanding. The initial interface is a CLI; the architecture is designed to migrate to a chat-based web UI in a future phase.

---

## 2. Goals

- Allow users to search for round-trip flights using natural language input from the terminal.
- Extract structured search parameters (origin, destination, dates, times, layover preferences) from free-form text.
- Query live flight data via SerpAPI's Google Flights integration (see Technical Considerations).
- Return ranked flight results with smart recommendations based on user preferences (price, duration, layovers).
- Lay a clean architectural foundation that can be extended to a web-based chat UI later.

---

## 3. User Stories

1. **As a traveler**, I want to type a plain-English description of my trip so that I don't have to manually fill out structured search fields.
2. **As a traveler**, I want to filter flights by departure time window (e.g., "morning flights only") so that I only see options that fit my schedule.
3. **As a traveler**, I want to specify a maximum number of layovers so that I can avoid overly complex itineraries.
4. **As a traveler**, I want to receive ranked recommendations (not just a raw list) so that I can quickly identify the best option for my needs.
5. **As a traveler**, I want the tool to ask me clarifying questions if my input is ambiguous so that I always get accurate results.

---

## 4. Functional Requirements

1. The system must accept natural language input from the user via the terminal.
2. The system must parse the input and extract the following structured parameters:
   - Origin airport/city
   - Destination airport/city
   - Outbound date (or date range)
   - Return date (or date range)
   - Preferred departure time window (e.g., morning, afternoon, evening) — optional
   - Preferred arrival time window — optional
   - Maximum number of layovers (0 = nonstop, 1, 2, any) — optional
   - Number of passengers — optional, defaults to 1
   - Cabin class (economy, business, first) — optional, defaults to economy
3. The system must prompt the user for any required missing parameters (origin, destination, outbound date, return date) before executing a search.
4. The system must query live flight data using the SerpAPI Google Flights endpoint.
5. The system must return a list of matching round-trip flight options including:
   - Airline(s)
   - Outbound and return flight numbers
   - Departure and arrival times for both legs
   - Number of stops and layover details
   - Total trip duration
   - Price (total, per person)
6. The system must rank and surface a "best pick" recommendation based on a combination of price, total duration, and number of stops.
7. The system must display results in a clean, readable format in the terminal.
8. The system must handle API errors gracefully and display a user-friendly error message.
9. The system must use **Pydantic AI** for agent orchestration, LLM interaction, and structured output validation.
10. The system must use **Pydantic models** to define and validate all flight search parameters and API response data.

---

## 5. Non-Goals (Out of Scope)

- One-way or multi-city trip searches (future phase).
- User accounts, saved searches, or search history.
- Price alerts or fare tracking over time.
- Booking or redirecting to a booking site.
- A web UI or chat interface (planned for Phase 2, not this phase).
- Support for hotels, car rentals, or any non-flight travel.
- Real-time seat maps or availability details.

---

## 6. Design Considerations

- **CLI UX:** The interaction should feel conversational — the agent should acknowledge what it understood before running the search (e.g., "Got it — searching round trips from JFK to LHR, June 20–27, nonstop…").
- **Output format:** Results should be printed in a structured, easy-to-scan table or card format in the terminal. Consider using the `rich` library for formatting.
- **Clarification flow:** If the user's input is missing required fields, the agent should ask one focused follow-up question at a time rather than dumping a form on the user.
- **Phase 2 readiness:** Agent logic, tool definitions, and data models should be kept separate from the CLI presentation layer so they can be reused in a web UI without rewriting core logic.

---

## 7. Technical Considerations

- **Language:** Python 3.11+
- **Core framework:** [Pydantic AI](https://ai.pydantic.dev/) — used for agent definition, tool use, LLM calls, and structured output.
- **Data models:** All flight search parameters and results must be defined as Pydantic `BaseModel` classes for validation and serialization.
- **Flight data source:** Google does not offer a public Google Flights API. The recommended approach is **SerpAPI** (`google_flights` engine), which provides structured Google Flights results via a paid API with a free tier. An alternative is the **Amadeus for Developers** API (free sandbox tier). The PRD uses "Google Flights data" to mean data sourced through one of these providers.
- **LLM backend:** Pydantic AI supports multiple LLM providers (OpenAI, Anthropic, Gemini, etc.). The specific model is left as a configuration choice; default to a capable, cost-efficient model (e.g., `claude-haiku` or `gpt-4o-mini`).
- **Environment variables:** API keys (SerpAPI, LLM provider) must be loaded from a `.env` file using `python-dotenv`. Keys must never be hardcoded.
- **Suggested project structure:**

  ```text
  flightFinder/
  ├── agent/
  │   ├── agent.py          # Pydantic AI agent definition
  │   ├── tools.py          # Flight search tool(s)
  │   └── models.py         # Pydantic data models
  ├── cli/
  │   └── main.py           # CLI entrypoint
  ├── .env.example
  ├── requirements.txt
  └── README.md
  ```

---

## 8. Success Metrics

- A user can type a natural language flight request and receive valid, ranked results within 10 seconds on a standard internet connection.
- The agent correctly extracts all required search parameters from a well-formed natural language input without needing follow-up clarification.
- The agent correctly identifies missing required fields and prompts the user for them before searching.
- Results match the filters specified by the user (correct dates, correct max layovers, correct cabin class).
- The codebase can be extended to a web UI (Phase 2) without rewriting the agent or tool logic.

---

## 9. Open Questions

All previously open questions have been resolved. Decisions are recorded below for reference.

| # | Question | Decision |
|---|----------|----------|
| 1 | Flight data provider | **SerpAPI** (Google Flights engine) |
| 2 | LLM backing the Pydantic AI agent | **Cheapest available model** (e.g., `claude-haiku-4-5` or `gpt-4o-mini`) — optimize for cost |
| 3 | Ranking / "best pick" logic | **Rule-based:** always surface the cheapest nonstop flight first |
| 4 | Date flexibility | **Exact dates only** — user will always provide specific outbound and return dates |
| 5 | Ambiguous city handling | **Not applicable** — user will always supply specific, unambiguous origin and destination locations |
