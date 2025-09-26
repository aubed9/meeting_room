# Project Requirements Document (PRD)

## Goals and Background Context

*This sharded file serves as the high-level summary of the project's purpose. The Scrum Master will use this to provide context to the development team as they begin their work.*

### Goals
* Increase meeting efficiency and eliminate wasted time from repeated discussions.
* Create a persistent, searchable, and time-lined history of all meeting discussions.
* Ensure clear, actionable tasks are extracted, assigned, and tracked through to completion.
* Provide managers with automated summaries and insights to improve future meetings.
* Establish a contextual knowledge base from meetings that can provide real-time assistance.

### Background Context
The project is initiated based on feedback from IT professionals who report that meeting quality is poor due to a lack of process and order. Key issues include discussions being forgotten, leading to repetition in future sessions, and the absence of structured outputs, leaving attendees unclear on their roles and responsibilities. This platform aims to solve these problems by introducing an agentic assistant to structure the meeting lifecycle.

---

## Requirements

### Functional (MVP Scope)
* **FR1 (Revised):** The system must capture audio from each meeting attendee through their browser via the web application.
* **FR2:** The system must use voiceprint technology to associate each audio stream with a specific, pre-registered user profile.
* **FR3:** The system must transcribe the captured audio into text (Speech-to-Text). (Note: For the demo, we assume a perfect transcript is provided).
* **FR4:** The system must generate a time-lined transcript of the meeting, classifying conversation points into main and sub-subjects.
* **FR5:** The system must analyze the transcript to extract and list potential tasks assigned during the meeting.
* **FR6:** A human supervisor must be able to review, edit, and approve the AI-extracted tasks before they are officially assigned to users.
* **FR8:** The meeting owner/host must have a UI control (e.g., a "Flag Conclusion" button) to signal to the AI that the current conversation point is a definitive decision-making section.
* **FR9:** The platform must provide users with clear guidelines and examples on how to phrase statements to maximize AI accuracy.
* **FR10:** The system must maintain a detailed user profile containing, at a minimum: full name, voiceprint, role in the company, projects involved, and a list of currently assigned tasks.
* **FR11:** The AI analysis agent must leverage the contextual information from attendee profiles (e.g., role, projects) to improve the accuracy and relevance of its meeting summary and task extraction.

### (Post-MVP) Future Enhancements
* The system will support online meetings through a cloud-based room.

### Non-Functional
* **NFR1:** The end-to-end latency from a person speaking to their transcribed text appearing in the timeline should be under 5 seconds for real-time features.
* **NFR2:** The speech-to-text transcription must achieve a Word Error Rate (WER) of less than 10% for standard business English in a quiet room environment.
* **NFR3:** All user data, including voice recordings and transcripts, must be encrypted both at rest and in transit.
* **NFR4:** The system must be able to process and analyze a continuous meeting of at least 2 hours in length.
* **NFR5:** The system must be designed to eventually support STT and text analysis for the Persian language.

---

## User Interface Design Goals

### Overall UX Vision
The user experience should be clean, efficient, and unobtrusive. For the web-based recording and review platform, the focus is on clarity and ease of navigation, enabling users to quickly capture audio, find insights, and manage tasks from complex meeting transcripts.

### Key Interaction Paradigms
* **Recording:** A "one-tap" interface to start and stop a recording. Visual feedback should clearly indicate that audio is being captured successfully.
* **Post-meeting review:** A dashboard-driven interface for accessing past meetings. The transcript view will be the core of the experience, with interactive elements to play back audio, view speaker tags, and see flagged tasks.

### Core Screens and Views
* Login / User Profile Screen (for voiceprint setup)
* New Meeting / Active Recording Screen
* Dashboard (list of past meetings)
* Meeting Results / Transcript View
* Task Review & Approval Screen (for supervisors)

### Accessibility
* **Standard:** WCAG 2.1 AA. The platform should be navigable and usable by people with disabilities, including screen reader support.

### Branding
* To be defined. The design should feel professional, modern, and trustworthy.

### Target Device and Platforms
* **Web Responsive:** The application will be optimized for desktop but functional on tablet and mobile browsers.

---

## Technical Assumptions

### Repository Structure
* **Choice:** Monorepo
* **Rationale:** This project has several distinct parts (web app, AI backend) that will need to share code (e.g., data types, API contracts). A monorepo makes managing this shared code and ensuring consistency across the entire project much simpler.

### Service Architecture
* **Choice:** Traditional Hosted Application
* **Rationale:** The application will be designed as a persistent, running service deployed on a dedicated host, such as Iranserver, as per project requirements.

### Testing Requirements
* **Choice:** Unit + Integration
* **Rationale:** To ensure a reliable and high-quality product, we need to test both the individual pieces of code (Unit tests) and how they work together (Integration tests). This is a standard practice for building robust applications.

### Additional Technical Assumptions and Requests
* The architecture must feature a strong context engineering and memory component. This system will be responsible for connecting agent outputs, user profile data, and historical meeting information to provide accurate, context-aware results for the AI analysis.

---

## Epic List (Phased Feasibility Demo)

*This file outlines the high-level development roadmap for our demo. It breaks the project into large, sequential phases, each delivering a significant piece of functionality. The Scrum Master will use this to guide the sprint-by-sprint development plan.*

* **Epic 1: Web-Based Audio Capture & Preprocessing**
    * **Goal:** Build a simple web application that allows authenticated users to record a meeting, flag key moments for tasking, and have the audio preprocessed (denoised and segmented by speaker using voiceprints) to create a standardized input for our analysis agent.
* **Epic 2: Core Analysis & Task Extraction**
    * **Goal:** Develop the first layer of the analysis engine. This agent will take the standardized, preprocessed data from Epic 1 and generate a structured, speaker-identified transcript and a clear list of tasks based on the user's flagged moments.
* **Epic 3: Advanced Analysis & Insight Generation**
    * **Goal:** Build the advanced analysis features. This agent will take the structured transcript from Epic 2 and generate the deep analysis timeline (with topics and a mindmap structure) and the overview section (summary, key takeaways, and AI advice for unsolved issues).
* **Epic 4: Dashboard & Visualization UI**
    * **Goal:** Build the user interface to display all the rich outputs. This includes creating the dashboard, the interactive timeline/mindmap view, the task list, and the complete overview section.

---

## Epic Details

*This is the most detailed part of our planning document. It breaks down each epic into specific, actionable stories with clear acceptance criteria. This file will be the primary reference for the Scrum Master and the development team during the sprints.*

### Epic 1: Web-Based Audio Capture & Preprocessing
**Expanded Epic Goal:** To build the foundational web application that can successfully capture and standardize our input data. This involves creating a simple UI for users to record a meeting, implementing the backend to handle the audio file, and running the preprocessing scripts to create a clean, speaker-segmented data file. This epic's output is the essential, high-quality data that the entire analysis pipeline will depend on.

**Stories:**

* **Story 1.1: Basic Web App & User Authentication**
    * As a developer, I want a basic web app with user registration and login, so that we have a secure foundation and can associate recordings with specific users.
    * **Acceptance Criteria:**
        * A monorepo with basic web and API applications is set up.
        * Users can register for an account and log in via the web UI.
        * A basic user profile is created in the database to store voiceprint data.

* **Story 1.2: Web Recording & Voiceprint UI**
    * As a user, I want to record my voiceprint and record a new meeting from my browser, so that I can capture meeting audio for analysis.
    * **Acceptance Criteria:**
        * A user can record and save their voiceprint on their profile page.
        * A "New Meeting" page allows a user to start and stop a recording using their browser's microphone.
        * The captured audio is sent to the backend API upon completion.

* **Story 1.3: Audio Preprocessing Pipeline**
    * As a system, I want a pipeline that automatically denoises and segments a recorded audio file by speaker, so that the audio is standardized for analysis.
    * **Acceptance Criteria:**
        * An API endpoint exists to receive the raw audio file from the web app.
        * A backend process is triggered that applies denoising algorithms to the audio.
        * The process then uses the user's saved voiceprint to segment the audio, identifying who spoke when.
        * The output is a structured data file containing the cleaned, speaker-segmented audio chunks.

* **Story 1.4 (Revised): Meeting Conclusion Flag**
    * As a meeting host, I want to flag the start and end of the decision-making part of the meeting, so that the AI knows exactly which section to analyze for actionable items.
    * **Acceptance Criteria:**
        * The "Flag Conclusion" button on the recording screen is a toggle (on/off).
        * When toggled on, the 'start' timestamp is recorded.
        * When toggled off (or the meeting ends), the 'end' timestamp is recorded.
        * An array of `{start, end}` timestamp pairs is saved with the meeting recording.

### Epic 2: Core Analysis & Task Extraction
**Expanded Epic Goal:** To build the first layer of our analysis engine. We will take the standardized, speaker-segmented data produced by Epic 1 and process it to generate a clean, structured transcript. We will also implement the first version of our task extraction logic, which will use the timestamps from the 'Conclusion Flag' to identify and list potential action items for review.

**Stories:**

* **Story 2.1: Ingest & Structure Transcript Data**
    * As a developer, I want to ingest the preprocessed data and a plain text transcript, so that I can create a single, structured data object for analysis.
    * **Acceptance Criteria:**
        * A service is created that takes the output from the Epic 1 pipeline (speaker segments, timestamps) and a text transcript as input.
        * The service correctly merges these inputs into a structured JSON object (e.g., `[{timestamp: "00:05", speakerId: "user_A", text: "..."}]`).
        * The final structured transcript is saved to the database, linked to the meeting.

* **Story 2.2 (Revised): Extract Conclusion Transcript Sections**
    * As a system, I want to extract the full transcript sections marked as the "conclusion," so that I have the complete context for task extraction.
    * **Acceptance Criteria:**
        * The service reads the structured transcript and the array of `{start, end}` timestamp pairs.
        * For each pair, it extracts the entire block of conversation from the transcript that falls between the start and end times.
        * The output is one or more large "conclusion" text blocks.

* **Story 2.3 (Updated Logic): AI-Powered Task Extraction**
    * As a developer, I want to send the large conclusion text blocks to an LLM to extract action items, so that we can automate task creation from the most relevant part of the meeting.
    * **Acceptance Criteria:**
        * The service takes a large "conclusion" text block as input.
        * The service uses a prompt to ask an LLM to identify all action items within the text.
        * The extracted tasks are structured and saved to the database in a "pending approval" state.

### Epic 3: Advanced Analysis & Insight Generation
**Expanded Epic Goal:** To build the advanced analysis features that differentiate our product. We will take the structured transcript from Epic 2 and generate the deep analysis timeline with a mindmap structure, and the overview section containing a summary, key takeaways, and AI-driven advice for any unresolved issues.

**Stories:**

* **Story 3.1: Topic Segmentation**
    * As a system, I want to analyze a transcript and automatically divide it into distinct topics of conversation, so that I can create a structured timeline.
    * **Acceptance Criteria:**
        * A service takes a structured transcript as input.
        * It uses an LLM or NLP techniques to identify shifts in conversation and group transcript lines into topics.
        * Each topic is given a concise, AI-generated title.
        * The transcript data is updated with these topic labels.
        * Topics must be substantive and distinct. A topic should represent at least 1-2 minutes of continuous conversation to avoid overly granular results.

* **Story 3.2: Dialogue Mindmap Generation**
    * As a system, I want to convert the dialogue within each topic into a mindmap data structure, so that users can easily visualize the flow of the conversation.
    * **Acceptance Criteria:**
        * A service takes the topic-segmented transcript as input.
        * For each topic, it generates a hierarchical structure (e.g., JSON) representing a mindmap of the key points, arguments, and counter-arguments.
        * The main topic is the central node, and related points are child nodes.
        * The mindmap data is saved and linked to the meeting.
        * The generated mindmap must have at least two levels of depth, clearly distinguishing between main points and supporting details.

* **Story 3.3: Summary & Key Takeaways Generation**
    * As a system, I want to generate a concise summary and a list of key takeaways from the meeting, so that users can quickly understand the most important outcomes.
    * **Acceptance Criteria:**
        * A service takes the full transcript as input.
        * It uses a specific LLM prompt to generate a one-paragraph executive summary.
        * It uses a separate LLM prompt to generate 3-5 bullet points of key takeaways.
        * The summary and takeaways are saved as part of the meeting's overview report.
        * The summary generation prompt must give higher weight to the conversation that occurred within the user-flagged "conclusion" section.

* **Story 3.4: Unsolved Issue Detection & AI Advisor**
    * As a system, I want to detect unsolved problems in the dialogue and suggest potential solutions, so that I can provide proactive advice.
    * **Acceptance Criteria:**
        * A service scans the transcript for phrases indicating an unsolved problem.
        * For each identified issue, it sends the relevant conversation context to an LLM.
        * The LLM is prompted to generate a neutral, helpful suggestion or a set of questions to guide the next discussion.
        * The "AI Advice" is saved as part of the meeting's overview report.
        * The "AI Advice" prompt must include the context of the speakers' roles and projects (from their profiles) to generate more relevant suggestions.
        * All AI-generated advice must be phrased as neutral questions or suggestions for further discussion, never as a definitive command or solution. A disclaimer, such as 'AI-generated suggestion for consideration,' must be included with the output.

### Epic 4: Dashboard & Visualization UI
**Expanded Epic Goal:** To build the user interface that brings all our backend work to life. We will create a web-based dashboard where a user can see their past meetings and view the rich, AI-generated outputs from Epic 3.

**Stories (Revised Order):**

* **Story 4.1: Meeting Results API Endpoint**
    * As a developer, I want a single API endpoint to fetch all analysis results for a specific meeting, so that the UI can easily display the information.
    * **Acceptance Criteria:**
        * A secure API endpoint (e.g., `GET /api/meetings/{meetingId}`) is created.
        * The endpoint returns a single JSON object containing all data for that meeting: the structured transcript, the list of tasks, the mindmap data, and the overview report content.

* **Story 4.2: Basic Meeting Dashboard**
    * As a user, I want to see a list of all my past meetings, so that I can easily access their results.
    * **Acceptance Criteria:**
        * After logging in, the user is shown a dashboard page.
        * The dashboard displays a list of the user's previously recorded meetings, showing the meeting title and date.
        * Each meeting in the list is a link to its dedicated results page.

* **Story 4.3: Task List & Approval UI**
    * As a supervisor, I want to see the list of pending tasks and be able to approve, edit, or delete them, so that I can finalize the meeting's action items.
    * **Acceptance Criteria:**
        * The meeting results page displays the list of "pending approval" tasks.
        * The supervisor has controls to edit the text of each task.
        * An "Approve" button changes a task's status from "pending" to "assigned."
        * A "Delete" button removes an incorrect task from the list.

* **Story 4.4: Interactive Transcript View**
    * As a user, I want to see the interactive transcript for a meeting, so that I can review the conversation.
    * **Acceptance Criteria:**
        * The meeting results page displays the full, time-lined transcript with clear speaker labels.

* **Story 4.5: Mindmap & Overview Visualization**
    * As a user, I want to see the meeting's mindmap and overview report, so that I can quickly understand the key points and advice.
    * **Acceptance Criteria:**
        * The meeting results page renders the mindmap data from the API into an interactive, visual diagram.
        * The UI displays the "Overview Section," including the AI-generated summary, key takeaways, and AI advisor suggestions.

---

## Success Metrics for the Demo

*This file is critically important as it defines the specific, measurable targets for our demo. We will use these metrics to objectively determine if our Proof of Concept is successful enough to warrant further development.*

1.  **Input Quality: Preprocessing**
    * **Speaker Identification Accuracy:** The system must correctly attribute over 95% of spoken sentences to the correct speaker in a test recording with 3-5 distinct, pre-registered speakers.
    * *Why this matters:* The quality of all downstream analysis depends on knowing who said what.

2.  **Core Analysis: Task Extraction**
    * **Task Extraction Recall:** The system must identify and correctly formulate at least 80% of the actionable tasks from the user-flagged 'conclusion' sections of three pre-scripted test recordings.
    * *Why this matters:* This is the primary "job" of the AI assistant and the core of the user's problem.

3.  **Advanced Analysis: Insight Quality**
    * **Insight Generation Usefulness (Qualitative):** In a review of the demo results, a human reviewer must rate the AI-generated Summary, Mindmap, and AI Advisor suggestions as "Helpful and Accurate" for at least 2 out of 3 test recordings.
    * *Why this matters:* This tests whether our advanced features are creating real value, not just noise.

4.  **Overall Performance**
    * **End-to-End Processing Time:** The total time from when a meeting recording is saved to when the full analysis is available on the dashboard must be less than 25% of the meeting's duration (e.g., a 60-minute meeting should be processed in under 15 minutes).
    * *Why this matters:* The tool needs to be timely to be useful.

---

## Checklist Results Report

*This file contains the summary of the final validation checklist run against the PRD. It confirms the document's readiness for the architecture and development phases.*

### Executive Summary
The PRD has been iteratively developed and is in excellent shape. The scope is well-defined for a feasibility demo, and the epic/story structure is logical and actionable. The initial gap regarding success metrics has been successfully addressed. The document is ready for the Architect to begin their work.

* **Overall PRD Completeness:** 95%
* **Demo Scope Appropriateness:** Just Right
* **Readiness for Architecture Phase:** High

### Category Analysis

| Category                       | Status | Notes                                           |
| ------------------------------ | ------ | ----------------------------------------------- |
| 1. Problem Definition & Context| PASS   |                                                 |
| 2. MVP Scope Definition        | PASS   |                                                 |
| 3. User Experience Requirements| PASS   |                                                 |
| 4. Functional Requirements     | PASS   |                                                 |
| 5. Non-Functional Requirements | PASS   | Key NFRs for a demo are established.            |
| 6. Epic & Story Structure      | PASS   | Iteratively refined to a logical, tech-first plan. |
| 7. Technical Guidance          | PASS   | Sufficient high-level assumptions for the Architect. |
| 8. Cross-Functional Requirements| PASS   | Key data models are implied and ready for architecture. |
| 9. Clarity & Communication     | PASS   |                                                 |

### Top Issues by Priority
**RESOLVED:** The initial high-priority issue was to define specific success metrics for the demo. This has been completed.

### Recommendations
The primary recommendation was to define the demo's success metrics, which has been successfully completed. The PRD is now considered complete and ready for the next phase.

---
