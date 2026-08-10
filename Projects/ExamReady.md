Repo: https://github.com/YapiSamuel/QuizGenerator

# 📝 ExamReady — Exam Composer for University Lecturers

**Bilingual (English / French). Runs entirely on the lecturer's own machine.**

ExamReady answers a question every lecturer meets at the end of a semester: *why does writing the exam mean retyping material you already taught?* A lecturer drops in their PowerPoint chapters, the application reads the slide text, drafts questions from it, and hands back a full editor — question type, wording, options, images, scoring — before exporting a print-ready paper laid out the way their university already formats exams. Written for a professor at my own institution, it is open source and remains a work in progress.

## 🔍 Features
- Parses `.pptx` decks in the browser with JSZip, reading slide text straight from the OOXML — lecture content never leaves the machine unless the lecturer explicitly asks for question generation
- Drafts multiple-choice, true/false and short-answer questions per chapter, written entirely in English or French regardless of the language of the source slides
- Full manual editor over every generated question: change type, rewrite text, add or remove options, mark correct answers, attach an image, or write questions from scratch
- Reproduces real exam-paper formatting — True/False as a V/F table or inline, checkbox or lettered markers, custom section titles, per-section scoring with negative marking, and numbering that runs continuously or restarts per chapter
- Takes the institutional header from an image or a Word document, converting `.docx` to HTML and sanitising it before it is stored or rendered
- Saves to SQLite so an exam can be reopened and revised weeks later, then exports to PDF for printing
- A local-only server holds the API key; the browser never receives a credential and never contacts the model provider directly, with an hourly quota and per-job token accounting so a runaway loop is caught before the invoice is

## 🧰 Skills Demonstrated
- **Letting the threat model choose the architecture:** the project began as a single 1,263-line HTML file and would have stayed there, except that a secret cannot be protected in a browser. That one fact required a backend; the backend made persistence practical; persistence is what made the tool useful. The security constraint was not a tax on the design — it *was* the design
- **Threat-modelling a change rather than a system:** adding a database converted reflected XSS into stored XSS. A malicious `.docx` header used to vanish on refresh; saved, it re-executes on every load, on an origin holding every unpublished exam. It is now sanitised on the way in *and* out, because trusting a future render path to clean stored HTML is how that bug survives a refactor
- **Recognising a boundary that looks safe and isn't:** with no login, the loopback bind *is* the authentication boundary — yet any site open in another browser tab can still reach `127.0.0.1`. That required Host pinning against DNS rebinding, an Origin allow-list, and a forced JSON content-type so a cross-site "simple request" cannot skip the CORS preflight
- **Defensive input handling:** files verified by magic bytes rather than the extension they claim, ZIP entry counts capped against decompression bombs, and an image allow-list that rejects SVG outright because it is an active-content format
- **Treating model output as hostile input:** every generated question is schema-validated server-side and re-coerced client-side, with bounded array lengths and string sizes, so a malformed response yields a smaller list rather than an exception
- **Verifying your own assumptions:** SQLite silently ignores `ON DELETE CASCADE` unless a pragma is set per connection, and does not auto-index foreign keys the way MySQL does. Reading `PRAGMA index_list` rather than my own schema file revealed three redundant indexes; query plans then proved the cascade still runs on an index
- **Refactoring for auditability:** splitting one file into 17 ES modules, with sanitisation confined to a single module, turned "escape everything" from a hope into a rule a reviewer can enforce — you cannot review what you cannot read
- Supply-chain hygiene: vendoring browser libraries instead of loading them from a CDN, which made a strict `script-src 'self'` policy possible and surfaced two critical advisories in a PDF dependency that a CDN tag would have hidden
- Stack: vanilla ES modules with no framework, Express 5, `node:sqlite`, Zod, DOMPurify, and the Anthropic API

## 🎯 Outcome
A working tool that turns an afternoon of exam writing into an hour of editing, built for one real user with one real workflow. What I value most is not the generation step but everything around it — deciding the browser gets no credential, noticing that persistence changed a vulnerability class rather than just adding a feature, and checking a live database instead of trusting the schema I had written myself. The habit it built was distrusting my own work: the redundant indexes, a middleware rule that blocked my own delete requests, and a vulnerable dependency were all found by testing rather than reading. Its limits are deliberate and documented — there is no authentication because there is exactly one user, and the server still treats its own client as the trusted renderer, which must change the day a second lecturer uses it. It is open source and still under active development.