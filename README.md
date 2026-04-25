# Certificate Transparency Proof Explorer

[ct.jordan-wright.com](https://ct.jordan-wright.com)

Trace a live HTTPS certificate into Certificate Transparency logs and verify its inclusion proof step by step.

<p align="center">
  <img src="web/og-image.png" alt="Certificate Transparency Proof Explorer preview" width="800">
</p>

Enter a site, fetch its live certificate, inspect its Signed Certificate Timestamps (SCTs), ask the matching CT log for an inclusion proof, and replay the Merkle audit path in the browser until it reaches the signed tree head.

## What It Shows

- The certificate chain returned by the target website.
- Normal TLS validation results, including hostname and chain checks.
- Embedded and TLS-delivered SCTs.
- The CT log that issued each SCT, using Chrome's public log list.
- The Merkle leaf hash used for the proof.
- The signed tree head root returned by the log.
- Every audit-path sibling hash.
- Every parent hash computed on the way from the certificate leaf to the tree root.

CT does not prove that a domain is safe or trustworthy. It proves that a specific Merkle tree leaf appears in a specific signed tree snapshot. The UI makes that boundary visible.

## How It Works

For a target like `https://jordan-wright.com`, the backend:

1. Opens a TLS connection to the target host.
2. Reads the leaf certificate and chain.
3. Parses SCTs from the certificate and TLS handshake.
4. Looks up SCT log IDs in Chrome's CT log list.
5. Rebuilds the CT Merkle leaf for the SCT.
6. Fetches the log's signed tree head.
7. Requests an inclusion proof for the leaf hash at that tree size.
8. Replays the audit path locally.
9. Returns a proof transcript for the frontend to visualize.

The backend uses Google's `certificate-transparency-go` library for CT protocol details. The local code focuses on producing a clear proof transcript that shows how the audit-path hashes combine into the signed root.

## Local Development

```sh
go run ./cmd/server
```

Then open:

```text
http://localhost:8080
```

Run tests:

```sh
go test ./...
```

## API

The local server and Lambda entrypoint expose the same endpoint:

```text
GET /api/analyze?url=https://example.com
```

The response includes certificate metadata, validation results, SCT details, CT log metadata, inclusion proof data, and the full audit-path hash transcript.

## Deployment

The frontend is static HTML, CSS, and JavaScript. The backend is Go and can run locally, as an AWS Lambda Function URL via `cmd/lambda`, or as a small Go service elsewhere.

To build a Linux ARM64 Lambda binary:

```sh
GOOS=linux GOARCH=arm64 go build -o bootstrap ./cmd/lambda
zip function.zip bootstrap
```

Use AWS Lambda's `provided.al2023` runtime with ARM64 architecture, upload `function.zip`, and enable a Function URL.
