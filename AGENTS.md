# AGENTS.md — instructions for coding agents

You are implementing **thaiid-go**. The roadmap, API design and acceptance criteria are in
`PLAN.md` — it is the source of truth. Read it fully before writing code.

## How to work
1. Pick the **first unchecked milestone** in PLAN.md (unless the human assigned one).
2. If the milestone says "Ben reviews/approves", produce the document, stop, and report — do not continue past it.
3. Write tests first from the spec/vectors, then the implementation.
4. Keep the public API exactly as designed in PLAN.md. If the design is wrong, change
   PLAN.md in the same PR and explain why in the PR description.
5. When the milestone's Definition of Done passes, tick its checkbox in PLAN.md with a one-line note
   and update the README if user-facing.
6. One milestone = one branch (`m<N>-<slug>`) = one PR.

## Commands
```sh
go vet ./...
go test -race ./...
golangci-lint run   # if installed
```

## Rules
- No new dependencies unless PLAN.md allows them. Core packages stay zero-dependency.
- No floats for money. No panics on user input. No network access in tests.
- Test vectors in `testdata/` are shared across repos — never edit them to make a test pass;
  if a vector is wrong, document the evidence in the PR.
- Never commit secrets, real personal data, real ID-card dumps or real certificates.
- Doc comments on every exported symbol; examples that compile.
- Commit messages: Conventional Commits.

## Sibling repos (same owner, same conventions)
thaiutils-go · thai_utils_dart · promptpay-go · thai_promptpay_dart · thaiid-go · thai-address · etax-go
