Roman Cole - Tech Manager
Mark Sanchez

Our hiring process consists of 5 steps:
1. Check the profile or CV
2. Intro Meeting (Hiring & Tech Manager)
3. Technical Meeting (CTO)
4. Final Interview (CEO and CFO)
5. Signing

https://calendly.com/mark_sanchez_tech/

UTC-05:00 Eastern Time (US & Canada)
UTC+03:00 Eastern European Time

8:00 -> 16:00
16:00 -> 24:00

Mark Sanchez
16:00 - 16:30, Friday, March 27, 2026

ROI - An annual Return on Investment (годовая доходность инвестиций)
CTO - A Chief Technology Officer (технический директор)
CEO - Chief Executive Officer (генеральный директор)
CFO - Chief Financial Officer (финансовый директор)
# Test Project 

[https://bitbucket.org/0xb2tirios/mvp/src/main/](https://bitbucket.org/0xb2tirios/mvp/src/main/)

## Task 1 — Simple Properties API in Go : 10 ~ 12 min

Goal : Show ability to build a clean, small REST service in Go.

What to do :
- Create a minimal Go service (e.g. go/property_api/).
- Define a Property struct (id, title, priceUSD, priceETH, location, roi, fundedPercent, status, etc.).
- Implement HTTP handlers (using net/http, Gin, Fiber, or Echo):
- GET /health → returns JSON { "status": "ok" }.
- GET /properties → returns a static slice of Property as JSON (hardcoded or loaded from a JSON file).
- GET /properties/{id} → returns a single property by id or 404 if not found.
- Use proper status codes and JSON encoding/decoding.
Deliverable : A small Go server that compiles and runs; in the video, start it and hit the endpoints.

## Task 2 — Property Filtering & Sorting Logic in Go : 10 ~ 12 min

Goal : Show domain logic and clean code structure.

What to do :
- In the same service or a separate package, implement pure functions that:
- Filter properties by:
- min/max price,
- min ROI,
- location substring.
- Sort properties by:
- price ascending/descending,
- ROI descending.
- Wire this into GET /properties:
- Accept query params like minPrice, maxPrice, minROI, location, sort.
- Apply filters and sorting in memory on the static slice.
Deliverable : Filtered/sorted responses; in the video, show a few example requests and how results change.

## Task 3 — Unit Tests for Handlers or Logic : 8 ~ 10 min

Goal : Demonstrate testing discipline in Go.

What to do :
- Add tests using Go’s standard testing package (and httptest if testing handlers).
- Choose one (or both):
- Option A (logic tests):
	- Test the filtering/sorting functions with small in-memory slices.
	- Cover at least:
	- Basic happy path,
	- No results,
	- Combined filters.
- Option B (handler tests):
	- Use httptest.NewRecorder and http.NewRequest (or framework equivalents) to:
		- Test GET /properties returns 200 and valid JSON,
		- Test GET /properties/{id} returns 404 for unknown id.
		- Make the tests deterministic and easy to read.
    
Deliverable : Passing go test in the module; in the video, run tests and briefly show one or two key assertions.