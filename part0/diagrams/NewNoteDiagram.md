``` mermaid
sequenceDiagram
    participant User
    participant Browser
    participant Frontend as JS
    participant Server
    participant DB

    User->>Browser: types note into text field and clicks Save
    Browser->>Frontend: form submit event (submit handler runs)
    activate Frontend
    note right of Frontend: handler prevents default form, reads input value and constructs JSON payload

    Frontend->>Server: POST https://studies.cs.helsinki.fi/exampleapp/notes
    Note right of Frontend: headers: Content-Type: application/json, body: { "content": "My new note" }
    activate Server

    Server->>DB: INSERT note (content, date)
    activate DB
    DB-->>Server: inserted row (id, created_at)
    deactivate DB

    Server-->>Frontend: 201 Created + JSON of created note
    deactivate Server

    alt Frontend appends created note to DOM
        Frontend-->>Browser: update DOM (append new note), clear input
    else Frontend re-fetches notes
        Frontend->>Server: GET https://studies.cs.helsinki.fi/exampleapp/data.json
        activate Server
        Server-->>Frontend: [{ "content":"...", "date":"..." }, ... ]
        deactivate Server
        Frontend-->>Browser: render notes list, clear input
    end

    Frontend-->>User: UI shows the saved note (confirmation)
    deactivate Frontend

---