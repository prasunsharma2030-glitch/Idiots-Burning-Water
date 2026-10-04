# Future Predictor (Python + Gemini)

1. Create a **new** Gemini key at aistudio.google.com, open `server.py`, and paste it between the quotes on the `API_KEY = ""` line at the very top. Save the file. (Setting `GEMINI_API_KEY` or using a `gemini-key.txt` file also works, but is optional.)
2. Run `py server.py` (or `python server.py`). No Node.js is required.
3. Open `http://localhost:3000` — do not open `site/index.html` directly.

The AI uses Gemini's OpenAI-compatible REST endpoint with `Authorization: Bearer ...`, which is the supported auth scheme for that endpoint. The default model is `gemini-3.5-flash-lite`, which has a much larger free daily quota than the full Flash models.

You can optionally set `GEMINI_MODEL` or `PORT` before starting the server.

**Security:** if you paste the key into `server.py`, never upload or commit that file. `gemini-key.txt` is already in `.gitignore`. Rotate any key that has been shared or exposed.

### AI usage
The chart and AI day score use one Gemini request together. The forecast is requested on initial load and when a forecast input changes; there is no periodic AI refresh timer.



AI request behavior
- AI chart + AI day score run only after an explicit forecast-value change.
- The live clock never calls Gemini.
- Duplicate control listeners were removed, so one change produces one graph request.
- Identical graph requests are cached by the local server for 6 hours.
- Gemini HTTP 429 responses trigger a cooldown and a built-in local fallback instead of repeated retries.


### If you see HTTP 429
429 means Gemini is limiting your requests. The server console prints which kind it was, and the page shows it too:
- **Daily limit** ("free daily request limit is used up"): free-tier quotas are per project and per model, they reset at midnight Pacific time, and Google can change them. Switch to a bigger-quota model (`set GEMINI_MODEL=gemini-3.5-flash-lite`, or `gemini-flash-lite-latest`), wait for the reset, or enable billing in Google AI Studio. Your exact limits are shown in AI Studio under your project's rate limits.
- **Per-minute limit**: wait about a minute. Several quick changes are now merged into one request (2 second wait, at least 3.5 seconds apart).
- **"Requests are coming too fast"**: that is the local server's own 3 second cooldown, not Google. It clears by itself.

The assistant panel no longer sends a request when the page opens. It only calls Gemini when you press a chip or send a message.

### Is the graph really using AI?
Only when the server is running and Gemini answers. Open `http://localhost:3000` (not the HTML file directly). If the AI can't be reached, the page falls back to its built-in formula and says so: the legend reads "Built-in forecast, AI is not being used" and the score box reads "Day score (built-in estimate)". The graph also stays on the built-in curve until you change an input, because loading the page or switching days does not call the AI.

### Accounts and saved records
Use **Account** in the sidebar to sign up or log in. Your daily records (sleep, steps, work, screen time, caffeine, water) and your whole color theme (primary background, secondary boxes, text color and preset) are saved to your account by calendar date and load again on any device you log in from, on any later day. Open Account to see your saved days and tap a recent one to jump to it. Everything lives in `data/users.json` next to `server.py`; keep that file private and out of version control (it is already in `.gitignore`).



AI forecast setup
------------------
When AI is configured, the dashboard shows “Waiting for AI forecast…” while the AI graph is being generated. When AI is not configured or the request fails, the built-in local graph remains visible and keeps working. Add your Gemini API key in `server.py` (`API_KEY = "..."`) or set `GEMINI_API_KEY` in the environment, then run `py server.py` and open `http://localhost:3000`. The default model is `gemini-3.5-flash-lite`.
