# ratify-verifier-go

[![Build Status](https://github.com/notaryproject/ratify-verifier-go/actions/workflows/build.yml/badge.svg)](https://github.com/notaryproject/ratify-verifier-go/actions/workflows/build.yml)
[![codecov](https://codecov.io/gh/notaryproject/ratify-verifier-go/branch/main/graph/badge.svg)](https://codecov.io/gh/notaryproject/ratify-verifier-go)
[![Go Reference](https://pkg.go.dev/badge/github.com/notaryproject/ratify-verifier-go.svg)](https://pkg.go.dev/github.com/notaryproject/ratify-verifier-go)

Go library providing verifier implementations for the [Ratify](https://github.com/notaryproject/ratify-go) verification framework. It enables cryptographic signature verification of OCI artifacts using industry-standard signing technologies.

## Verifiers

### Cosign

Verifies [Sigstore Cosign](https://docs.sigstore.dev/cosign/overview/) signatures on OCI artifacts, built on the [sigstore-go](https://github.com/sigstore/sigstore-go) library.

**Supported verification modes:**

| Mode | Description |
|------|-------------|
| **Keyless** | Verifies against Fulcio certificates, Rekor transparency logs, and OIDC identity policies. Ideal for CI/CD pipelines. |
| **Key-based** | Verifies against provided public keys. Supports custom key providers and validity periods. |

```go
import "github.com/notaryproject/ratify-verifier-go/cosign"
```

**Keyless verification example:**

```go
v, err := cosign.NewVerifier(&cosign.VerifierOptions{
    Name: "my-cosign-verifier",
    IdentityPolicies: []verify.PolicyOption{
        verify.WithCertificateIdentity(
            verify.NewShortCertificateIdentity("https://accounts.google.com", "", "user@example.com"),
        ),
    },
})
```

**Key-based verification example:**

```go
v, err := cosign.NewVerifier(&cosign.VerifierOptions{
    Name: "my-key-verifier",
    GetPublicKeys: func(ctx context.Context) ([]*cosign.PublicKeyConfig, error) {
        return []*cosign.PublicKeyConfig{
            {PublicKey: myPublicKey},
        }, nil
    },
})
```

For more details on Cosign concepts and verification workflows, see the [Cosign library documentation](./cosign/README.md).

### Notation

Verifies [Notation](https://notaryproject.dev/) signatures compliant with the CNCF Notary specification, built on the [notation-go](https://github.com/notaryproject/notation-go) library.

```go
import "github.com/notaryproject/ratify-verifier-go/notation"
```

**Example:**

```go
v, err := notation.NewVerifier(&notation.VerifierOptions{
    Name:           "my-notation-verifier",
    TrustPolicyDoc: trustPolicyDoc,
    TrustStore:     myTrustStore,
})
```

## Usage

All verifiers implement the `ratify.Verifier` interface from [ratify-go](https://github.com/notaryproject/ratify-go):

```go
result, err := verifier.Verify(ctx, &ratify.VerifyOptions{
    Store:              store,
    Repository:         "myregistry.io/myimage",
    SubjectDescriptor:  subjectDesc,
    ArtifactDescriptor: artifactDesc,
})
```

## Requirements

- Go 1.23 or later

## Build and Test

```sh
# Run all tests
make test

# Clean build artifacts
make clean
```

## Documentation

- [Cosign Library Documentation](./cosign/README.md) - Detailed Cosign/Sigstore concepts and verification workflows

## Related Projects

- [ratify-go](https://github.com/notaryproject/ratify-go) - Core Ratify verification framework
- [sigstore-go](https://github.com/sigstore/sigstore-go) - Sigstore verification library
- [notation-go](https://github.com/notaryproject/notation-go) - Notation signature verification library

## License

This project is licensed under the [Apache License 2.0](./LICENSE).
