# PLAN — thaiid-go

Status: **M0 scaffolded** · Owner: @Chawn · Last updated: 2026-10-06

## Goal
Read the Thai national ID smart card from any PC/SC reader in Go, and ship a small
**local bridge** (`thaiid-agent`) that exposes the card to web apps over
`http://127.0.0.1` / WebSocket. Government offices, hospitals, banks and clinics need
"เสียบบัตรแล้วกรอกฟอร์มให้" in browser-based systems; today most use closed vendor SDKs
(often Windows-only ActiveX/.NET). A cross-platform, auditable open-source option is the gap.

## Non-goals
- Writing to the card, PIN/biometric (match-on-card) operations, or anything requiring
  government-issued credentials/keys.
- Storing card data. The library returns data to the caller; it never persists or transmits it.

## Privacy & legal (put this in README, prominently)
- Card data is personal data under Thailand's **PDPA**. The library must not log personal
  fields at any log level. Tests use synthetic fixtures only — **never commit real card dumps**.
- The bridge binds to `127.0.0.1` only, requires an allow-listed `Origin` (CORS) and a
  per-session pairing token; reading the photo is opt-in per request.

## Card protocol (verify on real hardware — values below are the widely published ones)
- Transport: PC/SC via `github.com/ebfe/scard` (pcsclite on Linux/macOS, WinSCard on Windows).
- SELECT applet: `00 A4 04 00 08 A0 00 00 00 54 48 00 01`.
- READ command: `80 B0 <P1 P2 = offset> 02 00 <Le>` then GET RESPONSE `00 C0 00 00 <Le>`.
  Some card generations (ATR beginning `3B 67`) need GET RESPONSE `00 C0 00 01 <Le>` —
  select the variant from the ATR.
- Text fields are **TIS-620**; decode to UTF-8 (implement a tiny TIS-620 table, no dependency).
  Names are `#`-separated (prefix#first#middle#last).

| Field | Offset | Length |
|---|---|---|
| Citizen ID | 0x0004 | 0x0D |
| Name (TH) | 0x0011 | 0x64 |
| Name (EN) | 0x0075 | 0x64 |
| Date of birth (BE, YYYYMMDD) | 0x00D9 | 0x08 |
| Gender | 0x00E1 | 0x01 |
| Card issuer | 0x00F6 | 0x64 |
| Issue date | 0x0167 | 0x08 |
| Expiry date | 0x016F | 0x08 |
| Address | 0x1579 | 0x64 |
| Photo (JPEG) | 0x017B onward | 20 chunks × 0xFF |

## API (v0.1.0)
```go
package thaiid
type Card struct {
    CitizenID string
    NameTH, NameEN Name   // Prefix, First, Middle, Last
    BirthDate, IssueDate, ExpiryDate time.Time // converted from BE
    Gender Gender
    Issuer string
    Address Address       // HouseNo, Moo, Soi, Road, SubDistrict, District, Province (parsed from '#'-separated)
    Photo []byte          // only when ReadOptions.Photo is true
}
type ReadOptions struct{ Photo bool }
func ListReaders() ([]string, error)
func Read(ctx context.Context, reader string, opt ReadOptions) (*Card, error)
func Watch(ctx context.Context, reader string, opt ReadOptions) (<-chan Event, error) // insert/remove events

// Testability: everything above goes through an interface so tests use a fake card.
type Transmitter interface { Transmit(apdu []byte) ([]byte, error) }
func ReadFrom(t Transmitter, atr []byte, opt ReadOptions) (*Card, error)

// thaiid/tis620
func Decode(b []byte) string
```
Validate `CitizenID` checksum with `github.com/Chawn/thaiutils-go/thaiid` once that is released
(until then, a private copy of the 10-line function).

## `cmd/thaiid-agent` — local bridge
- `GET /v1/readers`, `GET /v1/card?photo=1`, `GET /v1/events` (WebSocket: inserted/removed/read).
- Config: allowed origins, port (default 127.0.0.1:8189 — check for conflicts), pairing.
- Builds: single static binary per OS via GoReleaser; Windows service / macOS launchd / systemd unit examples.
- A tiny JS client (`clients/js`, published to npm later) so web apps do `await thaiid.read()`.

## Milestones
- [x] **M0 — Scaffold**
- [ ] **M1 — TIS-620 decoder** + exhaustive table test.
- [ ] **M2 — APDU layer + fake card** (`ReadFrom` against a scripted Transmitter) — all parsing tested without hardware.
- [ ] **M3 — PC/SC integration** (`Read`, `ListReaders`, `Watch`); manual test checklist with real readers (document models tested).
- [ ] **M4 — CLI** `thaiid read --json`.
- [ ] **M5 — Agent** with security model above + JS client + example HTML page.
- [ ] **M6 — Release v0.1.0** with GoReleaser binaries (linux/amd64, arm64, windows, darwin).

## Definition of done
vet, race tests, lint; no personal data in fixtures or logs (add a test that greps logs);
README updated; PLAN ticked.

## Open questions
- Ben: which readers/card generations can you test with physically?
- Confirm offsets for the newest card generation before claiming support.
