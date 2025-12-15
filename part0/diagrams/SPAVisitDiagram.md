``` mermaid
sequenceDiagram
    participant User
    participant Browser
    participant CDN as "CDN / Static Server"
    participant SPA as "Frontend (SPA JS)"
    participant API as "API Server"
    participant DB

    User->>Browser: open https://studies.cs.helsinki.fi/exampleapp/spa
    Browser->>CDN: GET /exampleapp/spa (index.html)
    activate CDN
    CDN-->>Browser: HTML (index.html)
    deactivate CDN

    Browser->>CDN: GET /exampleapp/main.css
    activate CDN
    CDN-->>Browser: main.css
    deactivate CDN

    Browser->>CDN: GET /exampleapp/main.js
    activate CDN
    CDN-->>Browser: main.js
    deactivate CDN

    Note right of Browser: Browser parses HTML and executes JS, SPA bootstraps and client-side router initializes

    Browser->>SPA: run main.js -> mount app, client-side router
    activate SPA
    Note right of SPA: SPA renders initial shell and determines current route (/spa or /spa/notes)

    SPA->>API: GET /exampleapp/data.json (or GET /api/notes)
    activate API
    API->>DB: query notes
    activate DB
    DB-->>API: notes JSON
    deactivate DB
    API-->>SPA: 200 OK + [{ "content":"...", "date":"..." }, ...]
    deactivate API

    SPA-->>Browser: render notes list into DOM
    Note right of SPA: UI now shows notes, internal navigation uses client-side router (no full page reload)

    alt User navigates within the app (e.g. clicks "Create")
        User->>Browser: clicks internal link
        Browser->>SPA: client-side route change
        SPA-->>Browser: update DOM for new view (no network page reload)
    else User triggers data refresh (e.g. refresh button)
        SPA->>API: GET /exampleapp/data.json
        activate API
        API-->>SPA: updated notes JSON
        deactivate API
        SPA-->>Browser: re-render notes
    end

    deactivate SPA
```