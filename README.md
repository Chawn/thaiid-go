# thaiid-go

> **Status: pre-alpha — under active development. APIs will change until v0.1.0.**
> See [PLAN.md](PLAN.md) for the roadmap.

Read Thai national ID smart cards over PC/SC in Go, plus a local bridge so web apps can read the card.

อ่านบัตรประชาชนไทยผ่านเครื่องอ่าน Smart Card (PC/SC) ด้วย Go และมี agent ให้เว็บแอปเรียกอ่านบัตรผ่าน localhost ได้ — ข้ามแพลตฟอร์ม โค้ดเปิด ตรวจสอบได้

## Install
```sh
go get github.com/Chawn/thaiid-go
```

## Usage (target API)
```go
card, err := thaiid.Read(ctx, readerName, thaiid.ReadOptions{Photo: false})
fmt.Println(card.CitizenID, card.NameTH.First)
```

## Development
```sh
go vet ./...
go test -race ./...
golangci-lint run   # if installed
```

## Contributing
Issues and PRs welcome — see [CONTRIBUTING.md](CONTRIBUTING.md). Thai or English.

## License
MIT © Chawn and contributors
