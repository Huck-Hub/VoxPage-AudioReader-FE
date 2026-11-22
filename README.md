# VoxPage — Minimalist Document-to-Audiobook Studio (MEVN)

A **Full-Stack MEVN (MongoDB, Express.js, Vue.js 3, Node.js)** Document-to-Audiobook Reader inspired by **Speechify**, **ElevenLabs Reader**, and **Audible**, built 100% with the vanilla patterns from **CST3144 Full Stack Development**.

Paste any written document (or upload a `.txt` / `.md` file), and **VoxPage** automatically structures it into **Chapters**, **Pages**, and **Sentences**, reads it aloud continuously from beginning to end in **10+ languages**, lets you **skip to any chapter or page**, and **auto-saves your exact pause bookmark** (`currentChapterIndex`, `currentPageIndex`, `currentSentenceIndex`) to MongoDB and `localStorage` so you can leave and resume listening later.

---

## 1. Core Features & Step-by-Step Architecture

1. **Paste Written Text or Upload `.txt` / `.md` Files (`handleFileUpload()` & `createAndPlayDocument()`):**
   * Paste any essay, lecture notes, or book chapters into the document editor, or upload a local `.txt` / `.md` file using the browser's native `FileReader` API (`reader.readAsText(file)`).
   * Submits the new document via `POST /documents` to the Express.js & MongoDB backend (and backs it up to `localStorage`).
2. **Automatic Chapter & Page Splitter (`parseDocumentIntoChapters(rawText)`):**
   * Scans the document line-by-line for chapter headers (`Chapter 1`, `Part 1`, `Section 1`, or `# Heading`). If no headings are present, it automatically treats the text as `Chapter 1`.
   * Splits each chapter into individual sentences and groups every **3 sentences into a numbered Page** (`Page 1 of 4`, `Page 2 of 4`, etc.).
3. **Continuous Chapter-by-Chapter Audio Playback (`speakCurrentSentence()` & `advanceToNextSentence()`):**
   * Uses the browser's built-in **Web Speech API (`window.speechSynthesis` & `SpeechSynthesisUtterance`)**—requiring zero paid API keys.
   * When one sentence finishes speaking (`utterance.onend`), `advanceToNextSentence(true)` automatically moves to the next sentence on the page, flips to the next page when a page finishes, and advances to the next chapter until the entire book is read.
4. **Multi-Language & Voice Selector (`selectedLanguage` & `filteredVoices`):**
   * Choose from **10 languages** (`English UK en-GB`, `English US en-US`, `Spanish es-ES`, `French fr-FR`, `German de-DE`, `Arabic ar-SA`, `Portuguese pt-BR`, `Italian it-IT`, `Hindi hi-IN`, `Chinese zh-CN`) plus any system narrator voice installed on your computer or phone, and adjust playback speed (`0.75x` to `2.0x`).
5. **Skip to Any Chapter, Page, or Sentence (`jumpToChapter()`, `jumpToPage()`, `speakFromSentence()`):**
   * Click any **Chapter** in the left sidebar, pick any **Page number** from the **`Skip to Page (1 – N)`** dropdown, click **`Prev Page` / `Next Page`**, or **click directly on any sentence on the page** to jump the voice narrator to that exact line.
6. **Pause & Resume Later Bookmarking (`PUT /documents/:_id` + `localStorage`):**
   * Every sentence change and every click on **Pause & Save Bookmark** calls `saveBookmark()`, which updates `currentChapterIndex`, `currentPageIndex`, and `currentSentenceIndex` via `PUT /documents/:_id` on Express/MongoDB and saves to `localStorage` (`voxpage_docs_v1`).
   * When you return later, a warm gold **Saved Listening Bookmark Found** banner appears at the top so you can click **Resume Listening Here** to continue from the exact sentence where you stopped.

---

## 2. Researched 2026 UI/UX Theme: "Kuro-Shiro Warm Paper Dark Mode"

* **Deep Charcoal Paper (`#141210`) & Warm Stone Cards (`#1c1917`):** Eliminates blue-light eye strain and mimics reading a physical book under a warm desk lamp at night.
* **Literary Serif Reading Page (`Georgia` / `serif` at `1.15rem`, `1.85` line-height):** High-legibility cream text (`#e7e2dc`) with live **Warm Amber Gold (`#d97706`)** karaoke sentence highlighting (`active-speaking`).

---

## 3. Line-by-Line Code Explanation (In Simple Terms)

### A. Back-End (`backend/server.js`)
* **Lines 1–18 (Setup & MongoDB Connection):**
  * Imports `express`, `cors`, `path`, `fs`, `dotenv`, and `{ MongoClient, ObjectId }` from the native `mongodb` driver.
  * Loads `documents.json` (which contains 3 multi-chapter sample books) as an automatic fallback when `config.env` is not yet linked to MongoDB Atlas.
* **Lines 22–36 (Express Middlewares):**
  * `app.use(cors())` and `app.use(express.json())` allow the Vue front-end to send and receive JSON across origins.
  * **Logger Middleware** logs every request's HTTP method, URL, and timestamp to the terminal.
  * **Static File Middleware** serves `/images` from `staticFiles/` and returns a `404` if a file is not found.
* **Lines 48–137 (REST API Endpoints):**
  * `GET /documents`: Returns all saved audiobook documents from MongoDB (`db.collection("documents").find({}).toArray()`).
  * `GET /search?q=...`: Searches documents across `title`, `author`, `language`, and `rawText`.
  * `POST /documents`: Saves a newly pasted or uploaded document into MongoDB (`collection.insertOne(body)`).
  * `PUT /documents/:_id`: Saves the user's current listening bookmark (`currentChapterIndex`, `currentPageIndex`, `currentSentenceIndex`, `language`) using `$set`.
  * `DELETE /documents/:_id`: Deletes a document from the library.

### B. Front-End (`frontend/index.html`)
* **Document Parser (`parseDocumentIntoChapters(rawText)`, Lines 531–578):**
  * Splits raw text by lines, detects headings matching `/^(chapter\s+\d+|part\s+\d+|section\s+\d+|#{1,3}\s+)(.*)$/i`, splits chapter paragraphs into sentences (`/(?<=[.!?])\s+/`), and chunks every 3 sentences into a page array.
* **Web Speech API Engine (`speakCurrentSentence()`, Lines 722–757):**
  * Cancels any active speech (`window.speechSynthesis.cancel()`), creates `new SpeechSynthesisUtterance(sentence)`, assigns the selected language (`utterance.lang`), voice (`utterance.voice`), and reading speed (`utterance.rate`), and calls `window.speechSynthesis.speak(utterance)`.
  * Attaches `utterance.onend = () => this.advanceToNextSentence(true)` so the reader automatically steps through every sentence, page, and chapter in sequence.
* **Bookmark Sync (`saveBookmark()`, Lines 700–720):**
  * Saves `currentChapterIndex`, `currentPageIndex`, `currentSentenceIndex`, and `language` to both `localStorage` and `PUT /documents/:_id` so pausing is never lost on refresh.
