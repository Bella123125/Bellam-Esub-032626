
SmartMed Review 4.3 — Comprehensive Technical Specification (Streamlit on Hugging Face Spaces)
Document Version: 4.3.0
Date: 2026-03-26
Status: Final Technical Specification (Design + Implementation Guidance; no code included)
System Name: SmartMed Review 4.3 (智慧醫材審查指引與清單生成系統)
Deployment Target: Hugging Face Spaces (Streamlit)
Core Stack: Streamlit + agents.yaml + Multi-LLM routing (Gemini / OpenAI / Anthropic / Grok) + PDF/OCR toolchain
1. Executive Summary
SmartMed Review 4.3 is an advanced, AI-powered, agentic regulatory workspace designed strictly for medical device Regulatory Affairs (RA) teams, regulatory consultants, and official reviewers. The system acts as a multi-modal orchestrator, transforming fragmented regulatory texts, raw PDFs, and reviewer notes into highly structured, compliant Markdown documentation.
Version 4.3 builds upon the foundational architecture of 4.2, retaining 100% of all original capabilities—including the PDF preview and page selection, Python/Gemini OCR toolchains, dynamic agent orchestration via agents.yaml, SKILL panel rendering, and the highly praised WOW UI v2 layer (with its 20 painter-inspired styles and deep observability features).
The definitive upgrade in Version 4.3 is the introduction of the "Two-Phase Advanced 510(k) Guidance Transformer." This new pipeline allows users to seamlessly paste raw 510(k) submission summaries, review notes, or legacy guidance documents, and systematically mutate them through two distinct, human-in-the-loop phases. Phase 1 guarantees structural organization with strict constraints (5 tables, 20 contextual entities, 2000–3000 words), while Phase 2 applies a user-defined or system-default template to generate a massive, exhaustive 510(k) review guidance (3000–4000 words, 5 tables, 20 entities, and a review checklist), complete with iterative, model-selectable prompting capabilities.
1.1 Key Additions and Retained Features in 4.3
New: Two-Phase Advanced 510(k) Transformer Pipeline
Phase 1 (Organization): Ingests legacy 510(k) texts/notes and outputs exactly 2000–3000 words containing 5 tables and 20 contextual entities without losing original information.
Phase 2 (Template-Driven Synthesis): Merges the Phase 1 output with a specific template (custom or default) to produce a 3000–4000 word guidance document featuring exactly 5 tables, 20 entities, and a review checklist.
Iterative Refinement: Continuous prompting on the final output with full multi-model selection.
Retained: WOW UI v2 Layer
Theme toggle: Light / Dark
Language toggle: English / Traditional Chinese (繁體中文)
20 painter-inspired styles (selectable or randomized via the “Jackpot” function).
Retained: WOW Visualization Enhancements
WOW Interactive Indicator: Agent run states with drill-down storytelling.
Live Log: Streaming events, filtered searches, structured metadata, exportable.
WOW Interactive Dashboard: Cross-module analytics, run timelines, token/cost heuristics.
Retained: Agent Studio & AI Note Keeper
Step-by-step execution studio for agents.yaml orchestration.
AI Note Keeper featuring 9 distinct AI Magics, coral keyword highlighting, and continuous Markdown editing.
Retained: Environment-First Security & Standardized Model Allowlist
API keys sourced from Hugging Face environment variables; UI fallback is session-only.
Standardized LLM routing (OpenAI gpt-4o-mini, Gemini gemini-2.5-flash, Anthropic claude-3.5-sonnet, Grok grok-4-fast-reasoning, etc.).
2. Users, Use Cases, and Success Criteria
2.1 Primary Users
Regulatory Affairs (RA) Specialists: Responsible for drafting 510(k) submission strategies, comparing international regulations, and preparing evidence mapping.
Medical Device Reviewers: Tasked with triaging incoming submissions, identifying informational gaps, utilizing standard guidance templates, and drafting review memos.
Technical Leads / Prompt Engineers: Maintain the agents.yaml definitions, validate systemic prompts, and oversee deployment on Hugging Face Spaces.
2.2 Key User Journeys
The Advanced 510(k) Transformation (New in 4.3): A reviewer pastes a messy 510(k) submission summary. The system organizes it into a structured 2500-word Markdown document with exact entity extraction. The reviewer then applies the default "PTCA DCB" template. The system outputs a 3500-word comprehensive review guidance with checklists and tables. The reviewer refines the output via inline iterative prompting.
Standard Guidance Ingestion: Upload a legacy PDF, select specific pages, extract text via Gemini Vision OCR, and generate a standardized reviewer-friendly Markdown document with constraint checks (exactly 3 tables, 2000–3000 words).
Agent Pipeline Execution: Run sequential agents defined in agents.yaml, customizing the model and parameters at each step, and manually editing the output before passing it to the downstream agent.
AI Note Keeper: Paste unstructured meeting notes, organize them with coral-highlighted keywords, and apply one of 9 AI Magics (e.g., Traceability Matrix Builder or Contradiction Detector).
2.3 Definition of “Excellent Results”
Absolute Information Retention: The new Phase 1 transformer must demonstrably retain all factual claims, metrics, and nuances from the user's input while reshaping the formatting.
Strict Constraint Adherence: Length constraints (word counts), table counts (exactly 5 for the new pipeline, exactly 3 for the legacy pipeline), and entity extraction counts (exactly 20) are treated as binding contracts by the prompt engineering layer.
Explainability and Observability: Every LLM call is logged. The WOW Dashboard accurately reflects token usage, bottlenecks, and generation parameters.
Seamless Human-in-the-Loop: Users are never locked out of the generation process. Output from every single step is presented in a dual-pane text/Markdown editor before proceeding.
3. Scope, Non-Goals, and Design Principles
3.1 In-Scope Modules
WOW Dashboard, Interactive Indicator, and Live Logs.
Standard Guidance Workspace (PDF upload, OCR, 3-table/2500-word generation).
New: Advanced 510(k) Guidance Transformer (Phase 1 & Phase 2, 5-table/20-entity generation, iterative prompting).
AI Note Keeper (9 AI Magics, coral highlighting).
Agent Studio (agents.yaml orchestration).
SKILL panel rendering (SKILL.md).
Multi-provider model routing.
3.2 Non-Goals (Explicit)
No Code Generation: This specification dictates design and architecture only.
No Persistent Databases: There is no multi-user authentication, RBAC, or persistent PostgreSQL/MongoDB backend. All state is ephemeral and bound to the Streamlit session.
No Deterministic Guarantee: The system uses probabilistic LLMs. Constraint heuristics (like exact word counts) are "best-effort" enforced via prompt engineering and post-generation validation warnings, but mathematical perfection is not guaranteed.
No Legal Authority: Outputs are strictly drafting aids and do not constitute official regulatory clearance.
3.3 Design Principles
Transparency First: If an OCR process fails or a prompt is blocked due to a missing API key, the system must explain why in the user's selected language.
Graceful Degradation: If optional components (like PyTesseract for local OCR) are unavailable in the Hugging Face environment, the system seamlessly falls back to base PDF text extraction or Gemini Vision.
Unified Aesthetic: The WOW UI v2 design tokens (spacing, typography, painter style themes) must be rigorously applied to both new and legacy modules.
4. High-Level Architecture
4.1 Runtime Components
A. Streamlit Presentation Layer
Sidebar: Houses global configurations including Light/Dark theme toggles, Language selection (EN/ZH-TW), Painter Style selector (20 styles + Jackpot), API Key manager (fallback), and global model defaults.
Main Content Area: Tabbed interface separating the Standard Guidance Workspace, the new Advanced 510(k) Transformer, Agent Studio, Note Keeper, and WOW Dashboard.
Global Header: Persistent WOW Interactive Indicator, run-state storytelling, and quick-action buttons (Export Logs, Clear Session).
B. Ephemeral Session State Layer
Stores raw uploaded bytes, rendered PyMuPDF image thumbnails, extracted text arrays, editable Markdown artifacts, pinned prompts, and continuous iterative prompt chat histories.
Maintains structured run events for the Live Log module.
C. Document Processing & OCR Layer
Text Extraction: pypdf for native digital text.
Image Rendering: PyMuPDF and Pillow for converting PDF pages to processable images.
Local OCR: pytesseract (environment permitting).
Cloud OCR: Gemini Vision for complex, multi-column, or scanned legacy regulatory documents.
D. Multi-Provider LLM Routing Layer
A unified adapter pattern routing generation requests to OpenAI (Chat Completions), Gemini (generate_content), Anthropic (Messages API), or Grok (xAI REST).
Handles standardized inputs (System Prompt, User Prompt, Temperature, Max Tokens) and outputs across all providers.
E. Agent Orchestration Layer
Parses the agents.yaml file to dynamically render multi-step UI workflows.
Facilitates the passing of the "effective output" (user-edited text) from Agent N to Agent N+1.
5. WOW UI v2 (Theme, Language, Painter Styles, Jackpot)
5.1 Global Environmental Controls
The WOW UI v2 layer ensures that the analytical complexity of regulatory review is offset by an aesthetically pleasing, highly legible interface.
Theme: Users can toggle between Light and Dark modes. The system automatically adjusts font colors, background contrasts, and shadow depth to maintain WCAG accessibility standards.
Language Switcher: English / Traditional Chinese (繁體中文). This affects UI labels, button texts, error messages, and Live Log narrative strings. (Content generation language is dictated by the prompt, not the UI language).
Jackpot Function: A randomized selection algorithm that picks one of the 20 Painter Styles, applying a cohesive color and typography overlay to the entire session.
5.2 The 20 Painter Styles
The system incorporates 20 distinct design token sets inspired by historical art styles. These tokens dictate the background gradient palette, accent color harmonies, panel border glow intensities, and button styling (solid, glassmorphism, or outlined).
Monet: Soft atmospheric blues and greens; frosted glass panels.
Van Gogh: High contrast yellow and deep blue; vibrant active states.
Picasso: Geometric block layouts; sharp, contrasting borders.
Klimt: Golden ornamental accents; rich, warm typography highlights.
Hokusai: Indigo wave gradients; clean, flat minimalist buttons.
Rothko: Expansive color fields; subtle, soft-edged UI components.
Pollock: Energetic, speckled progress bars; chaotic but contained accents.
Vermeer: Calm, directed light gradients; high focus on the active text editor.
Matisse: Bold, high-contrast cut-out shapes for tags and badges.
Rembrandt: Chiaroscuro effect (ideal for dark mode); deep shadows and bright focal points.
Turner: Misty, glowing transitional animations between agent states.
Cézanne: Structured, earthly depth; distinct layering of UI cards.
Magritte: Surreal clarity; ultra-minimalist text with unexpected accent colors.
Dalí: Dream distortion; slightly skewed active state borders (subtle CSS transforms).
Kandinsky: Abstract rhythm; dynamic, colorful charts in the Dashboard.
Hopper: Quiet modernism; muted, isolated panel designs.
O’Keeffe: Organic minimalism; smooth, rounded corners and soft coral accents.
Basquiat: Street annotation; raw monospace fonts for labels, stark contrasts.
Bruegel: Dense detail; optimized spacing for high-information-density tables.
Ukiyo-e: Woodblock flatness; elegant, muted pastels with sharp, dark text.
6. WOW Visualization Enhancements
The observability suite is mandatory across all modules (including the new 510(k) Transformer).
6.1 WOW Interactive Indicator
Located in the persistent header, this component reflects the global session state (Idle, Running, Blocked, Warning, Error, Success).
Drilldown Mechanism: Clicking the indicator opens a drawer detailing exactly what the LLM is doing (e.g., "Phase 2: Applying Default PTCA Template... Elapsed Time: 14s").
It provides immediate actions such as "Cancel Run" (interrupting the API stream if supported) or "View Live Log".
6.2 Streaming Live Log
A dedicated panel providing structured, real-time observability into the agentic processes.
Event Schema: Every LLM call, OCR page completion, and preflight check emits an event containing a timestamp, run UUID, module name, severity, and token size estimates.
Privacy Guarantee: The log dynamically redacts substrings matching API key formats.
Export: Users can export the session's log as a .jsonl or .txt file for technical debugging.
6.3 WOW Interactive Dashboard
A holistic control center visualizing session productivity.
Run Timeline: A Gantt-like chart of all generations and OCR tasks.
Model Mix: A donut chart showing the proportion of usage between OpenAI, Gemini, Anthropic, and Grok.
Constraint Compliance Monitor: A dedicated widget tracking how often the LLM successfully hits the strict table counts (e.g., 3 tables vs 5 tables) and word count targets.
Token/Cost Estimator: A heuristic-based panel showing estimated prompt and completion tokens used during the session.
7. API Key Management (Environment-First)
Security and ease of use are prioritized through a two-tier API key handling architecture.
7.1 Key Sources and Priority
Environment Secrets: The system strictly prefers Hugging Face Space secrets (OPENAI_API_KEY, GEMINI_API_KEY, ANTHROPIC_API_KEY, XAI_API_KEY). If these exist, the UI absolutely never displays them. The API Key panel simply shows a green "Managed by environment" badge.
Session UI Fallback: If an environment key is missing, the sidebar renders a password-masked input field. Keys entered here are bound exclusively to the ephemeral Streamlit session state and are purged upon refresh.
7.2 Preflight Validation
Before any request is sent to an LLM, a preflight check runs:
Validates the selected model against the available keys.
If the key is missing, the WOW Indicator shifts to Blocked, and a localized error message instructs the user on how to provide the necessary credentials.
8. Standardized Multi-Provider Model Selection
All generation modules share a unified model selection dropdown, ensuring consistency and allowing the user to match the model to the task's complexity.
OpenAI: gpt-4o-mini, gpt-4.1-mini (optimized for strict constraint following).
Gemini: gemini-2.5-flash, gemini-3-flash-preview (excellent for massive context windows and OCR).
Anthropic: claude-3.5-sonnet, claude-3.5-haiku (superior for nuanced regulatory tone and markdown formatting).
Grok / xAI: grok-4-fast-reasoning, grok-3-mini (fast reasoning and data extraction).
9. Legacy Modules: Guidance Workspace & AI Note Keeper
To satisfy the requirement of retaining all original features, the following modules remain fully intact and upgraded with the WOW UI v2 layer.
9.1 Standard Guidance OCR & Generator (Step 1 - 4)
Ingestion & OCR: Users upload a PDF, preview thumbnails, select pages, and extract text via pypdf, pytesseract, or Gemini Vision OCR.
Generation: Converts the raw text into a reviewer-friendly Markdown document.
Strict Constraints: Must be exactly 2000–3000 words. Must contain exactly 3 Markdown tables.
Post-Generation: User evaluates compliance via heuristics and can manually edit the Markdown/TXT output before downloading.
9.2 Agent Studio (agents.yaml)
A stepwise execution environment where users load predefined prompts, bind inputs/outputs, select models per step, and dynamically edit the output of Agent N before it feeds into Agent N+1.
9.3 Enhanced AI Note Keeper (9 AI Magics)
Users paste messy meeting notes or personal text. The base agent organizes the text and applies coral highlighting to key terms. The user can then apply one of 9 AI Magics:
Executive Summary Builder
Action Items Extractor
Risk & Mitigation Draft
Meeting Minutes Formatter
Regulatory Checklist Generator
Keyword Highlighter/Refiner
Traceability Matrix Builder (Maps claims to evidence types with uncertainty tags).
Contradiction & Ambiguity Detector (Scans for conflicting regulatory statements).
Regulatory-Ready Rewrite (Transforms casual notes into localized formal submission tone).
10. NEW: Two-Phase Advanced 510(k) Review Guidance Transformer
This entirely new, premier module is designed to bridge the gap between messy, unstructured 510(k) applicant submissions (or reviewer notes) and highly formalized, template-driven regulatory guidance documents. It operates in two explicit phases, followed by an iterative refinement loop.
10.1 Phase 1: Source Document Organization & Entity Extraction
The Objective: Cleanse, structure, and normalize raw 510(k) data without losing a single piece of original information, while strictly formatting the output to specific analytical parameters.
1. Input Mechanism:
A massive text area where the user can paste any combination of 510(k) submission summaries, reviewer notes, or legacy 510(k) review guidance in raw TXT or Markdown format.
Preflight checks ensure the input is not entirely empty and does not exceed the maximum token context of the selected model.
2. Transformation Rules & Constraints (System Prompt Directives):
Zero Data Loss: The LLM is explicitly prompted to retain all original factual information, numerical data, test results, and claims.
Length Target: The output must be organized into a comprehensive Markdown document ranging between 2000 to 3000 words.
Table Constraint: The document must feature exactly 5 structured Markdown tables summarizing the core data (e.g., Performance Testing, Material Composition, Biocompatibility).
Entity Constraint: The document must identify and define exactly 20 Key Entities with Context. These entities (e.g., specific device components, biological markers, regulatory definitions) must be formatted distinctly (e.g., bolded terms followed by a contextual definition block).
3. Output & User Control:
The resulting text is rendered in a dual-pane editor (Markdown Preview alongside Raw Text Editor).
The user has total freedom to modify the output, correct entity definitions, or adjust tables before passing the document to Phase 2.
10.2 Phase 2: Template-Driven Guidance Synthesis
The Objective: Take the highly organized text from Phase 1 and morph it into a formal, structured 510(k) Review Guidance document mapped exactly to a specified template.
1. Input Mechanism (The Template Selector):
The user must define the target structure. They can:
Paste a custom 510(k) review guidance template.
Provide a plain-text description of how the guidance should be structured.
Select the Default Template.
2. The Default Template Definition:
If the user selects the default template, the system injects the exact following structure into the LLM's system prompt:
經皮冠狀動脈擴張術 (PTCA) 藥物塗層球囊 (DCB) 導管審查指引
1. 範疇
本指引適用於用於經皮冠狀動脈擴張術 (PTCA) 的藥物塗層球囊 (DCB) 導管。這些裝置設計用於在球囊擴張過程中，將藥物直接傳遞至冠狀動脈壁，以改善管腔直徑並降低再狹窄的發生率。
1.1 分類與法規狀態
TFDA 分類：E.0005 (經皮冠狀動脈擴張術導管)
風險等級：第三級 (高風險)
排除產品：含有生物藥物成分的組合產品不在本指引範疇內
1.2 基本原則
製造商必須提供完整的驗證數據，涵蓋臨床前測試以確保安全性與有效性。所有測試必須在完成、滅菌的產品或經驗證的等效樣品上進行。若使用非滅菌或未完成的樣品，必須提供充分的科學理由。
2. 審查重點
2.1 技術文件與產品描述
審查人員必須確認中文使用說明書 (IFU/標籤) 包含：
導管與藥物塗層的詳細材料組成 (含賦形劑)
藥物含量與劑量密度 (µg/mm²)
尺寸規格：球囊直徑、長度、工作長度、相容導絲尺寸
物理性能指標：額定爆破壓力 (RBP)
儲存條件：溫度、光敏性、保存期限
2.2 藥物成分的化學、製造與管制 (CMC)
已核准藥物：若藥物來源、製程、規格與台灣已核准藥物相同，可免除 CMC 資料，僅需提供證明
藥品主檔案 (DMF)：若藥物有有效 DMF，需提供授權書 (LOA) 與相關摘要
新藥物質：需提交完整 CMC 資料，包括外觀、聚合物分析、鑑別、雜質、殘留溶劑、含量均勻性
2.3 製造流程
重點在「塗層製程」：
提交詳細製造流程圖，標示關鍵管制點
最終產品規格與分析證書 (COA)
驗證塗層均勻性 (縱向與環向)
3. WOW 1：美人魚流程圖 (審查流程)
生物成分？ -> 標準 DCB？ -> 是 → 否 → 開始：上市前申請 -> 技術文件審查 → 藥物成分評估 → 臨床前工程測試 → 安全性與有效性 → 最終核准決策
4. 安全與性能數據要求
4.1 工程與功能測試
球囊額定爆破壓力 (RBP)、疲勞測試 (反覆充氣/放氣)、順應性 (壓力 vs. 直徑)、接合強度 (導管各部件連接處)、柔韌性與抗折性能
4.2 塗層特性
劑量密度、塗層完整性 (模擬血管追蹤前後)、顆粒評估 (脫落顆粒數量與大小)、體外藥物釋放曲線
4.3 藥理與毒理
新藥物質需完整非臨床測試 (GLP)、一般毒性 (兩種哺乳動物)、安全藥理 (心血管、呼吸、中樞神經)、基因毒性與生殖毒性、藥代/藥效學 (PK/PD)
5. WOW 2：國際法規比較 (表 1)
特徵 | 台灣 (TFDA) | 美國 (FDA) | 歐盟 (MDR)
分類 | 第三級 (高風險) | 第三級 (PMA) | 第三級 (規則 8/14)
審查途徑 | 臨床前指引 (105.01.14) | 上市前核准 (PMA) | MDR 2017/745 附錄 IX/X
藥物成分 | 需 CMC；允許 DMF | 組合產品 (CDRH/CDER) | 與 EMA/國家機構協商
臨床數據 | 視情況；常需本地 | 必須 (IDE/PMA) | 必須 (臨床評估)
生物相容性 | ISO 10993 系列 | ISO 10993 + FDA 指引 | ISO 10993 系列
上市後監測 | 定期安全報告 | 年度報告/上市後研究 | PSUR / PMCF
6. WOW 3：模擬拒件風險評估 (表 2)
缺失類型 | 風險等級 | 風險描述 | 預測/緩解措施
CMC 不足 | 高 | 缺乏藥物塗層穩定性數據或雜質分析 | 驗證藥物保存期限符合裝置儲存條件
顆粒數量 | 高 | 顆粒脫落超過 USP <788> 標準 | 優化塗層製程並進行模擬測試
等效性缺口 | 中 | 使用非滅菌樣品測試無科學理由 | 所有性能測試需在完成滅菌產品上進行
PK/PD 缺口 | 中 | 劑量密度高卻未評估全身暴露 | 提供動物 PK 數據比較局部與全身濃度
老化方案不足 | 中 | 加速老化未包含塗層完整性與藥物含量分析 | 將所有保存期限項目納入老化驗證
7. WOW 4：法規追溯熱圖 (表 3)
測試類別 | 測試項目 | 參考標準 | TFDA 重點
生物相容性 | 細胞毒性/致敏性 | ISO 10993-5 / ISO 10993-10 | 主要接觸材料
血液相容性 | ISO 10993-4 | 與血液成分交互作用
滅菌/致熱源 | EO 殘留/內毒素 | ISO 10993-7 / USP <85> | 滅菌保證水準 (SAL 10⁻⁶)
機械性能 | 額定爆破壓力 (RBP) | ISO 25539-1 / FDA PTCA 指引 | 統計信心 (95/95)
導管接合強度 | ISO 10555-1 / TFDA 指引 | 接合部拉伸強度
塗層 | 總藥物含量 | USP <905> / ICH Q6A | 劑量均勻性
顆粒物 | USP <788> / ASTM F2119 | 栓塞風險評估
藥理 | 一般毒性 | ICH S2B / S7A | GLP 合規
包裝 | 完整性/運輸壓力 | ISO 11607 / ASTM D4169 | 無菌屏障維持
8. 技術深度解析
8.1 藥代動力學與藥物釋放
測量球囊表面殘留藥物
3. Transformation Rules & Constraints for Phase 2:
Length Target: The synthesized output must strictly be between 3000 and 4000 words. This massive expansion requires the LLM to extrapolate reviewer expectations, detail engineering methodologies, and provide deep analytical context based only on the facts provided in Phase 1.
Table Constraint: Exactly 5 Markdown tables must be generated within the text (aligning with the template requirements such as the International Comparison, Risk Assessment, and Traceability Heatmap).
Entity Constraint: Exactly 20 Entities with Context must be embedded into the guidance document, ensuring specialized terminology is immediately defined for the reviewer.
Checklist Generation: The LLM must append a comprehensive Review Checklist at the end of the document, summarizing actionable items for the regulatory reviewer (e.g., Checkbox for "Verify RBP meets 95/95 statistical confidence").
4. Output Presentation:
Rendered in the dual-pane editor. The WOW Indicator validates the constraints (Word Count ~3500, Tables = 5, Entities = 20). If heuristics fail, a soft Warning state is triggered, allowing the user to click "Regenerate with enforced constraints".
10.3 Phase 3: Iterative Refinement & Prompting (Chat-on-Result)
Because drafting a 4000-word regulatory guidance is complex, single-shot generation is rarely perfect. Phase 3 introduces a localized Chat UI directly beneath the Phase 2 output.
Functionality: Users can keep prompting on the results. For example, typing "Expand section 4.2 to include more details on particulate shedding standard USP 788" and pressing Send.
Model Flexibility: Before sending the refinement prompt, the user can switch the underlying model (e.g., from Gemini to Anthropic Claude 3.5 Sonnet) to utilize different reasoning capabilities for the revision.
State Updating: The LLM receives the current state of the document, applies the user's instructions, and updates the text editor with the revised Markdown. Users can manually type over the result at any time.
11. Reliability, Guardrails, and Error Handling
11.1 Fallbacks and Recovery
If cloud APIs experience rate limiting (HTTP 429), the system intercepts the error, updates the Live Log with a severity of "Warning," and displays a localized WOW Indicator drawer explaining the throttle with a suggested backoff time.
If Phase 1 text exceeds the maximum context length of the chosen model, the preflight module blocks the request and suggests switching to a model with a larger context window (e.g., Gemini 2.5 Flash).
11.2 Hallucination Guardrails
Medical device regulation requires strict adherence to facts. System prompts in the new 510(k) Transformer explicitly enforce: "Do not fabricate clinical data, engineering standards, or regulatory pathways. If information mapping to the template is missing from the source text, insert a placeholder marked as [REQUIRES VERIFICATION] and state the uncertainty clearly."
11.3 Constraint Compliance Checking
The system employs post-generation Python heuristics (regex counting for |---| to detect tables, regex for bolded definition blocks for entities, whitespace/character counting for word estimates).
Failure to meet the 5-table or 20-entity constraint does not crash the app; it flags a UI Warning and recommends manual correction or a targeted iterative prompt via Phase 3.
12. Security, Privacy, and Data Handling
12.1 Ephemeral by Design
The Hugging Face Spaces environment utilizing Streamlit is inherently stateless between cold boots.
SmartMed Review 4.3 relies purely on st.session_state. If the user closes the tab or refreshes the page, all uploaded PDFs, extracted notes, generated guidance documents, and UI-entered API keys are permanently destroyed.
12.2 Output Portability
Given the lack of backend storage, all major modules (Guidance Workspace, Agent Studio, Note Keeper, 510(k) Transformer) feature prominent Download buttons, allowing users to export their artifacts as .md or .txt files directly to their local machines.
13. Deployment and Configuration
13.1 Hugging Face Spaces Architecture
Framework: Streamlit (Python 3.10+).
Files: app.py (entry point), agents.yaml (agent definitions), SKILL.md (skills panel content), requirements.txt (dependencies including pypdf, pytesseract, pymupdf, provider SDKs).
Environment Setup: Administrators must configure Space Secrets for the required LLM providers to ensure seamless, keyless operation for end users.
13.2 Resource Management
CPU/Memory Limits: Standard Hugging Face Spaces containers possess limited RAM. The system limits concurrent PyMuPDF image rendering to prevent out-of-memory (OOM) exceptions. Deep iterative generation loops rely on remote cloud APIs, keeping local CPU usage light.
14. Acceptance Criteria (Version 4.3)
WOW UI Integration: The system successfully loads and applies the Light/Dark theme, EN/ZH language toggles, and all 20 historical painter styles seamlessly across legacy and new modules.
Observability Verification: The WOW Indicator correctly tracks generation phases. The Live Log captures real-time preflight and completion events. The Dashboard accurately tallies session activity.
Legacy Feature Retention: Standard Guidance PDF processing works as in 4.2. OCR workflows function. The AI Note Keeper successfully executes all 9 Magics with coral highlighting. Agent Studio parses agents.yaml without error.
New Phase 1 Functionality: The user can paste a 510(k) summary; the selected LLM returns an organized Markdown document constrained strictly to 2000–3000 words, containing exactly 5 tables and exactly 20 contextual entities. All source data is retained.
New Phase 2 Functionality: The user can apply the explicit Default PTCA DCB Template. The LLM merges the Phase 1 output with the template to produce a 3000–4000 word guidance document. It must contain exactly 5 tables, 20 entities, and a review checklist.
Iterative Refinement: The user can interact with an inline chat box below the Phase 2 output to sequentially refine the text using any provider model from the allowlist.
Security Guarantee: Environment API keys are never leaked to the UI or logs; user-entered keys are purged upon session termination.
