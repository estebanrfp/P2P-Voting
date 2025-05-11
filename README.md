# P2P Real-time Voting App (P2P-Voting)

![P2P Voting App Screenshot](https://i.imgur.com/ERP8CAk.png) <!-- Replace placeholder.png with an actual screenshot -->

This is a minimalist, responsive, real-time peer-to-peer (P2P) voting application built entirely in a single HTML file, showcasing the capabilities of [gdb-p2p](https://github.com/estebanrfp/gdb), a minimalist graph database with P2P support.

**Live Demo:** [https://estebanrfp.github.io/P2P-Voting/]

## 🌟 Core Idea

The goal is to create a decentralized voting system where users can:
1.  **Create new polls** with a name, proposal options, and an end time.
2.  **Share a unique link** for others to vote.
3.  **View active polls** and their countdowns.
4.  **Vote on proposals** in real-time.
5.  **See results update instantly** across all connected peers.
6.  **Delete polls** they've created (or manage them if extended with ownership).

All of this happens without a central server, leveraging the P2P nature of `gdb-p2p` for data storage and synchronization (though full P2P sync between different browser instances/devices requires explicit peer configuration not implemented in this basic demo).

## ✨ Advantages of GraphDB (`gdb-p2p`) Showcased

This application demonstrates several key advantages of using `gdb-p2p`:

1.  **Decentralization & P2P:**
    *   The app can function without a central server. Data for polls (Voting Sessions) and their Proposals are stored and potentially synchronized peer-to-peer. While this example primarily uses the local browser storage (IndexedDB via `gdb-p2p`), the underlying database is designed for P2P.
    *   **Impact:** Reduces reliance on single points of failure and control, potentially increasing censorship resistance and data ownership.

2.  **Real-time Querying & Updates:**
    *   `gdb-p2p`'s `.map()` method with a callback function allows the app to listen for changes in the database in real-time.
    *   **Impact:**
        *   **Live Vote Counts:** When a user votes, the `votes` count on a `Proposal` node is updated. All other connected clients subscribed to that poll's proposals see the vote count change instantly without needing to refresh.
        *   **Dynamic Active Polls List:** The sidebar listing active polls updates automatically as new polls are created or existing ones end/are deleted.
        *   **Live Countdown Updates:** Countdowns for poll end times are managed client-side, but changes to a poll's status (e.g., from 'active' to 'ended') are reflected in real-time.

3.  **Simple Data Modeling (Graph Structure):**
    *   Data is naturally represented as nodes and (implicit) relationships:
        *   `VotingSession` nodes: Store poll name, end time, status.
        *   `Proposal` nodes: Store proposal titles and vote counts, linked to a `VotingSession` via a `sessionId` property.
    *   **Impact:** While `gdb-p2p` is "minimalist" and doesn't enforce explicit link/edge nodes in its core API for this example, the concept of connected data is inherent. Querying proposals *for a specific session* (`query: { type: "proposal", sessionId: "XYZ" }`) is straightforward.

4.  **Ease of Use & Rapid Development:**
    *   The API (`put`, `get`, `map`, `remove`) is concise and easy to understand.
    *   Integration via CDN makes it simple to get started in a single HTML file.
    *   **Impact:** Allowed for quick prototyping and development of a feature-rich P2P application.

5.  **Local-First Potential:**
    *   Data is stored locally in the browser's IndexedDB. The app can function offline for polls already loaded. With P2P sync configured, it could reconcile changes when peers reconnect.
    *   **Impact:** Improved performance and offline capabilities.

## 🚀 Features Implemented

*   **Poll Creation:**
    *   Set a Poll Name/Topic.
    *   Interactively add/remove proposal options (titles only for simplicity).
    *   Set a specific end date and time for the poll.
    *   Generate a unique shareable link (based on the poll's `sessionId`).
*   **Poll Listing (Sidebar):**
    *   Displays a list of all "active" polls.
    *   Shows the name/topic of each poll.
    *   Shows a real-time countdown for each active poll.
    *   Updates dynamically as polls are created or finished.
    *   Allows deletion of polls (with confirmation).
*   **Voting Interface:**
    *   Displays the selected poll's name/topic.
    *   Shows a main real-time countdown for the selected poll.
    *   Lists all proposal options for the current poll.
    *   Displays current vote counts for each proposal, updating in real-time.
    *   Allows users to vote **once per poll** (tracked via `localStorage`).
*   **Real-time Updates:**
    *   Vote counts update live across clients viewing the same poll.
    *   The list of active polls updates live.
*   **Results:**
    *   Once a poll ends (countdown reaches zero), voting is disabled.
    *   The winning proposal(s) are highlighted.
*   **Responsive Design:**
    *   The layout adapts to different screen sizes, with a focus on using available space efficiently and minimizing unnecessary scroll.
    *   Creator view uses a two-column layout on larger screens, stacking on mobile.
    *   Voting view uses a sidebar and main content area, stacking on mobile.
*   **Client-Side Logic:**
    *   The entire application runs in the browser in a single HTML file.
    *   Data persistence through `gdb-p2p` (IndexedDB).

## 🛠️ Tech Stack

*   **HTML5**
*   **CSS3** (including CSS Grid for layout)
*   **JavaScript (ES Modules)**
*   **[gdb-p2p](https://github.com/estebanrfp/gdb)**: The star of the show! Minimalist Graph Database with P2P support and real-time querying.

## ⚙️ How it Works (Simplified)

1.  **Poll Creation:**
    *   User inputs poll details.
    *   A `votingSession` node is created in `gdb-p2p` with a unique ID, name, end time, and `status: "active"`.
    *   For each proposal option, a `proposal` node is created, linked to the `votingSession` via its `sessionId`.
2.  **Sharing:**
    *   The URL hash (`#sessionId`) is used to share and load specific polls.
3.  **Viewing & Voting:**
    *   When a user opens a poll link, the app fetches the `votingSession` and its associated `proposal` nodes.
    *   `db.map()` with a callback is used to listen for real-time updates to proposals (vote counts) and the list of active sessions.
    *   When a vote is cast:
        *   The app checks `localStorage` to prevent repeat voting in the same session.
        *   The `votes` property of the chosen `proposal` node is incremented using `db.put()`.
        *   `localStorage` is updated to mark that the user has voted in this session.
        *   All subscribed clients see the vote count update.
4.  **Poll Ending:**
    *   Client-side countdowns manage the timing.
    *   When a poll's `endTime` is reached, its `status` in the `gdb-p2p` database is updated to `"ended"`. This change is picked up by other clients, disabling voting and showing results.
5.  **Poll Deletion:**
    *   User confirms deletion.
    *   The app first queries for all `proposal` nodes linked to the `votingSession`.
    *   Each `proposal` node is removed using `db.remove(proposalId)`.
    *   The main `votingSession` node is removed using `db.remove(sessionId)`.
    *   UI updates to reflect the deletion.

## 🚀 Potential Future Enhancements

*   Explicit P2P peer connection setup for true multi-device/browser sync.
*   User authentication/identity (e.g., using cryptographic key pairs) for more robust "vote once" mechanisms and poll ownership.
*   Editing existing polls.
*   More advanced query/filtering for polls.
*   Storing vote attributions (who voted for what, if privacy allows).
*   Improved UI/UX with a dedicated frontend framework.

## 🏗️ Setup & Running

1.  Clone this repository (or just save the HTML file).
2.  Open the `[your-filename].html` file in a modern web browser.
    *   To test real-time updates easily, open the same poll link in two different tabs or windows of the *same browser*.

That's it! No build steps or complex dependencies are needed for this basic version.
## License

This example project is for demonstration purposes. If based on a specific repository, refer to its license. Otherwise, consider it under a permissive license like MIT if you are distributing it.

[P2P-Voting Demo](https://estebanrfp.github.io/P2P-Voting/) Powered by [GraphDB (GDB)](https://github.com/estebanrfp/gdb)

-------------

#### Credits
[by Esteban Fuster Pozzi (estebanrfp)](https://github.com/estebanrfp)
