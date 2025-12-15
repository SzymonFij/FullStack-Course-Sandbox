``` mermaid
sequenceDiagram
    participant User
    participant Browser
    participant Static as "Static Server (SPA bundle)"
    participant SPA as "Frontend (SPA JS)"
    participant API as "API Server"
    participant DB

    User->>Browser: open https://studies.cs.helsinki.fi/exampleapp/spa
    Browser->>Static: GET /exampleapp/spa  (single initial request for the SPA)
    activate Static
    Static-->>Browser: index.html + bundled JS/CSS (single payload)
    deactivate Static

    Note right of Browser: Browser parses HTML and executes the bundled JS, SPA bootstraps and client-side router initializes

    Browser->>SPA: run bundle -> mount app
    activate SPA
    Note right of SPA: SPA renders shell and may fetch initial data via XHR/fetch

    SPA->>API: GET /exampleapp/data.json  (XHR)
    activate API
    API->>DB: query notes
    activate DB
    DB-->>API: notes JSON
    deactivate DB
    API-->>SPA: 200 OK + [{ "content":"...", "date":"..." }, ...]
    deactivate API

    SPA-->>Browser: render notes list into DOM

    User->>Browser: types note into text field and clicks Save
    Browser->>SPA: form submit event handled by SPA (preventDefault)
    note right of SPA: handler reads input and constructs JSON payload

    SPA->>API: POST /exampleapp/notes  (XHR: Content-Type: application/json, body: {"content":"My new note"})
    activate API

    API->>DB: INSERT note (content, date)
    activate DB
    DB-->>API: inserted row (id, created_at)
    deactivate DB

    API-->>SPA: 201 Created + JSON of created note
    deactivate API

    alt optimistic update
        SPA-->>Browser: append new note to DOM immediately, clear input
    else update after response
        SPA-->>Browser: update DOM with created note from response, clear input
    end

    SPA-->>User: UI shows the saved note (confirmation) — no page reload
    deactivate SPA
```