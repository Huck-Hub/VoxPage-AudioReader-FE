# VoxPage — Universal Document & Multimedia Audiobook Studio (MEVN)

A **Full-Stack MEVN (MongoDB, Express.js, Vue.js 3, Node.js)** Document-to-Audiobook & Multimedia Reader inspired by **Speechify**, **ElevenLabs Reader**, and **Audible**, built 100% with the vanilla patterns from **CST3144 Full Stack Development**.

Upload or paste **any document format** — **Word (`.docx`)**, **PDF (`.pdf`)**, **PowerPoint (`.pptx`)**, **Text / Markdown (`.txt`, `.md`)**, or **Video files (`.mp4`, `.webm`, `.ogg`)** — and **VoxPage** automatically:
1. Extracts the text and structures it into **Chapters / Slides / PDF Pages** and **Sentences**.
2. Extracts and plays any **embedded video clips** inside your Word (`.docx`) or PowerPoint (`.pptx`) files, PDF video links, or directly uploaded video files right inside the built-in **Document Video Theater**.
3. Reads the document aloud continuously from beginning to end in **10+ languages**, lets you **skip to any chapter or page**, and **auto-saves your exact pause bookmark** (`currentChapterIndex`, `currentPageIndex`, `currentSentenceIndex`) to MongoDB and `localStorage` so you can leave and resume listening later.

---

## 1. Universal File & Embedded Video Extraction (How It Works)

1. **Word Documents (`.docx` via Mammoth.js + JSZip):**
   * `mammoth.extractRawText({ arrayBuffer })` extracts all paragraphs and chapter headings from the `.docx` file.
   * Simultaneously, `JSZip.loadAsync(arrayBuffer)` inspects the internal `word/media/` folder inside the `.docx` archive. If your Word document contains an embedded video (`.mp4`, `.webm`, `.ogg`, `.mov`), it extracts the binary video into a playable browser `Blob` URL (`URL.createObjectURL(blob)`) and attaches it to the document's video player.
2. **PDF Files (`.pdf` via PDF.js):**
   * Loops through every PDF page (`1` to `pdf.numPages`) using `pdf.getPage(p)` and `page.getTextContent()`.
   * Automatically creates a dedicated chapter for each PDF page (`PDF Page 1`, `PDF Page 2`, etc.) and scans `page.getAnnotations()` for any embedded video hyperlinks (`.mp4`, YouTube, etc.).
3. **PowerPoint Presentations (`.pptx` via JSZip + XML DOMParser):**
   * Opens the `.pptx` ZIP archive, finds all slide XML files (`ppt/slides/slide1.xml` to `slideN.xml`) sorted in presentation order, and extracts every text node (`<a:t>`).
   * Each PowerPoint slide becomes a selectable chapter (`Slide 1`, `Slide 2`, ...), and any video embedded on a slide inside `ppt/media/` is automatically extracted and made playable in the **Video Found in This Document** player!
4. **Built-in Document Video Theater (`<video>` & `<iframe>` Player):**
   * Whenever a document contains an embedded video (from a `.docx` or `.pptx`), an uploaded video file, or a video URL in the text, a glowing **"Video Found in This Document"** banner appears above the reading page.
   * Clicking **"Play Video in Document"** opens the video player and automatically pauses the voice narrator (`onVideoPlay()`) so the narrator and video never talk over each other.

---

## 2. Core Audiobook Features

* **Continuous Chapter-by-Chapter Audio Playback (`speakCurrentSentence()` & `advanceToNextSentence()`):**
  * Uses the browser's built-in **Web Speech API (`window.speechSynthesis` & `SpeechSynthesisUtterance`)**.
  * Automatically advances sentence-by-sentence, page-by-page, and chapter-by-chapter from beginning to end.
* **Multi-Language & Voice Selector (`selectedLanguage` & `filteredVoices`):**
  * Choose from **10 languages** (`English UK en-GB`, `English US en-US`, `Spanish es-ES`, `French fr-FR`, `German de-DE`, `Arabic ar-SA`, `Portuguese pt-BR`, `Italian it-IT`, `Hindi hi-IN`, `Chinese zh-CN`) plus any system narrator voice and reading speed (`0.75x` to `2.0x`).
* **Skip to Any Chapter, Slide, Page, or Sentence (`jumpToChapter()`, `jumpToPage()`, `speakFromSentence()`):**
  * Click any **Chapter / Slide / PDF Page** in the left sidebar, pick any **Page number** from the **`Skip to Page (1 – N)`** dropdown, or **click directly on any sentence on the page** to jump the voice narrator to that exact line.
* **Pause & Resume Later Bookmarking (`PUT /documents/:_id` + `localStorage`):**
  * Saves `currentChapterIndex`, `currentPageIndex`, and `currentSentenceIndex` to Express/MongoDB and `localStorage` (`voxpage_docs_v2`) so you can close the tab and click **Resume Listening Here** whenever you come back.
