# Chat with any GitHub Repo

Sign in with GitHub, pick a repository, and ask questions about its code in plain English. Answers stream in live and link back to the exact files and line ranges they came from.

> "Where is authentication handled?" · "Explain what this function does" · "How does the data flow from the API to the UI?"

I built this as a hands-on way to learn how **Retrieval-Augmented Generation (RAG)**, **embeddings**, **OAuth**, and **streaming responses** work together in a real full-stack app.

## Features

- **GitHub OAuth login** with the access token kept server-side only
- **Repo picker** that lists your most recently updated repos
- **Automatic indexing** that fetches files, filters out junk (`node_modules`, lock files, build output), splits code into overlapping chunks, and embeds them
- **Semantic search** that finds the code most relevant to your question, even when your words don't match the code
- **Streaming answers** delivered token by token over Server-Sent Events
- **Cited sources**, with every answer linking to `file:line-range` on GitHub
- **Grounded prompting** so the model is told to answer only from retrieved code and admit when it can't find something

## How it works

```
 INDEXING (once per repo)
 GitHub API -> filter files -> split into chunks -> embed each chunk -> vector store.e

 ASKING (every question)
 question -> embed -> find top-6 similar chunks -> build prompt with those chunks
          -> LLM streams answer -> UI shows text + file citations
```

1. **Fetch.** The server reads the repo's file tree through the GitHub API and downloads source files (capped at 150 files under 50 KB each).
2. **Chunk.** Each file is split into ~40-line windows with a small overlap, so every chunk keeps its file path and line numbers.
3. **Embed.** Each chunk is converted into a vector using `text-embedding-3-small`.
4. **Retrieve.** Your question is embedded the same way and compared against all chunks by similarity. The top 6 are selected.
5. **Generate.** Those chunks go into a prompt, and the LLM streams its answer back to the browser via SSE.

## Tech stack

| Part | Technology |
|---|---|
| Backend | Node.js, Express |
| Auth | GitHub OAuth 2.0, `express-session` |
| Embeddings and LLM | OpenAI API (`text-embedding-3-small`, `gpt-4o-mini`) |
| Vector store | In-memory (cosine similarity via dot product) |
| Streaming | Server-Sent Events |
| Frontend | Vanilla HTML, CSS and JavaScript |



## Project structure

```
repo-chat/
├── server.js          # OAuth, GitHub fetching, chunking, embeddings, retrieval, streaming chat
├── public/
│   └── index.html     # Chat UI (login, repo picker, streaming messages, citations)
├── .env.example       # Template for required environment variables
└── package.json
```

## Known limitations

- The vector store is in memory, so indexes are lost when the server restarts
- Chunking is line-based and not aware of function or class boundaries
- Large repos are capped at 150 files to keep indexing fast and cheap
- No chat memory yet, so each question is answered independently
- The OAuth scope is `repo` so private repos work, which is broader than a public-only app needs

## Roadmap

- [ ] Persist embeddings in a real vector DB (Chroma, Qdrant, or pgvector)
- [ ] Smarter chunking with tree-sitter (split by function and class)
- [ ] Conversation history so follow-up questions work
- [ ] Syntax-highlighted code blocks and markdown rendering in answers
- [ ] Background indexing with a progress bar
- [ ] Re-index only changed files
- [ ] Rebuild the frontend in React
- [ ] Deploy (Render or Railway)

## What I learned

- How the OAuth authorization-code flow works end to end, and why the token must never reach the browser
- Why embeddings enable search by meaning rather than keywords
- How chunk size and overlap affect answer quality
- How to reduce hallucinations by grounding the prompt in retrieved context
- How to stream LLM output with SSE and parse it incrementally on the client

## License

MIT
