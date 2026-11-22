# VoxPage — Universal Document & Audiobook Studio (MEVN)

A production-ready **Full-Stack MEVN (MongoDB, Express.js, Vue.js 3, Node.js)** Document-to-Audiobook & Multimedia Reader built with the vanilla patterns from **CST3144 Full Stack Development**.

Import or paste **Word (`.docx`)**, **PDF (`.pdf`)**, **PowerPoint (`.pptx`)**, **Text / Markdown (`.txt`, `.md`)**, or **Video files (`.mp4`, `.webm`, `.ogg`)**, and **VoxPage**:
1. Displays the exact **Interactive Table of Contents** right beside the active reading page—showing every Chapter / Section / Slide / PDF Page, its starting **Book Page number (`p. 1`, `p. 4`, ...)**, and every nested sub-page with live audio progress tracking.
2. Lets the user **click any chapter or sub-page in the Table of Contents** (`playFromTocChapter()` / `playFromTocPage()`) to immediately jump there and begin playing audio from that exact section.
3. Reads the entire book aloud continuously in **10+ languages**, extracts and plays any **embedded video clips** inside `.docx`, `.pptx`, or `.pdf` files, and **auto-saves your exact pause bookmark** to MongoDB (`PUT /documents/:_id`) and `localStorage`.

---

## 1. Live Interactive Table of Contents & Global Book Pagination

* **`tableOfContents` Computed Property (`frontend/index.html`):**
  * Automatically maps every extracted chapter, section, PDF page, or PowerPoint slide into a structured Table of Contents entry with:
    * `chapterIndex` and exact heading `title` (`Chapter 1: ...`, `Capítulo 1: ...`, `Slide 1: ...`, `PDF Page 1`, etc.)
    * `startGlobalPage`: Exact starting page number across the whole document (`p. 1`, `p. 3`, `p. 6`).
    * `subPages`: Every individual page inside that chapter with a text preview and global page reference (`p. 1`, `p. 2`, ...).
* **Click-to-Play from the Table of Contents (`playFromTocChapter()` & `playFromTocPage()`):**
  * Clicking any chapter row or sub-page row in the Table of Contents sidebar immediately moves the playhead to that section, updates the persistent bookmark, and starts reading aloud (`speakCurrentSentence()`).
* **Global Book Page Jumper (`Go to Book Page 1 – N`):**
  * Lets the listener jump directly to any page number across the entire book (`jumpToGlobalPage()`).

---

## 2. Universal Document & Embedded Video Extraction

1. **Word Documents (`.docx` via Mammoth.js + JSZip):**
   * Extracts all paragraphs and headings via `mammoth.extractRawText({ arrayBuffer })` and extracts any embedded video files from `word/media/` via `JSZip`.
2. **PDF Documents (`.pdf` via PDF.js):**
   * Extracts text page-by-page (`1` to `pdf.numPages`) and detects any video annotations (`page.getAnnotations()`).
3. **PowerPoint Presentations (`.pptx` via JSZip + XML `DOMParser`):**
   * Extracts slide titles and body text from `ppt/slides/slide1.xml` .. `slideN.xml` in order, plus any embedded slide videos from `ppt/media/`.
4. **Built-in Document Media Player (`<video>` & `<iframe>`):**
   * Automatically pauses the voice narrator whenever the user opens or plays an embedded document video (`openAndPlayVideo()` / `onVideoPlay()`).
