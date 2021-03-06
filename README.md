# jwt (JSON Web Token for Go)
[![JWT Compatible](https://jwt.io/img/badge.svg)](https://jwt.io)

[![Build Status](https://travis-ci.org/gbrlsnchs/jwt.svg?branch=master)](https://travis-ci.org/gbrlsnchs/jwt)
[![GoDoc](https://img.shields.io/badge/godoc-reference-blue.svg)](https://godoc.org/github.com/gbrlsnchs/jwt)

## About
This package is a JWT signer, verifier and validator for [Go] (or Golang).

## Usage
Full documentation [here].

## Example (from `example_test.go`)
```go
package jwt_test

import (
	"context"
	"fmt"
	"net/http"
	"net/http/httptest"
	"time"

	"github.com/gbrlsnchs/jwt"
)

func Example() {
	// Timestamp the exact moment this function runs
	// for validating purposes.
	now := time.Now()
	// Mock an HTTP request for showing off token extraction.
	w := httptest.NewRecorder()
	r := httptest.NewRequest(http.MethodGet, "/", nil)
	// Build JWT from the incoming request.
	jot, err := jwt.FromRequest(r)

	if err != nil {
		// Handle malformed token...
	}

	if err = jot.Verify(jwt.HS256("secret")); err != nil {
		// Handle verification error...
	}

	// Define validators for validating the JWT. If desired, there
	// could be custom validators too, e.g. to validate public claims.
	algValidator := jwt.AlgorithmValidator(jwt.MethodHS256)
	audValidator := jwt.AudienceValidator("test")
	expValidator := jwt.ExpirationTimeValidator(now)

	if err = jot.Validate(algValidator, audValidator, expValidator); err != nil {
		switch err {
		case jwt.ErrAlgorithmMismatch:
			// Handle "alg" mismatch...

		case jwt.ErrAudienceMismatch:
			// Handle "aud" mismatch...

		case jwt.ErrTokenExpired:
			// Handle "exp" being expired...
		}
	}

	// "Sign" issues a raw string, but if desired, one could also
	// use "FromString" method to have a JWT object.
	token, err := jwt.Sign(jwt.HS256("secret"), &jwt.Options{Timestamp: true})

	if err != nil {
		// ...
	}

	auth := fmt.Sprintf("Bearer %s", token)

	w.Header().Set("Authorization", auth)
	w.WriteHeader(http.StatusOK)
	w.Write([]byte(token))
}

func ExampleFromContext() {
	jot, err := jwt.FromString("eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.e30.t-IDcSemACt8x4iTMCda8Yhe3iZaWbvV5XKSTbuAn0M")

	if err != nil {
		// Handle malformed token...
	}

	key := "JWT"
	ctx := context.WithValue(context.Background(), key, jot)
	jot, err = jwt.FromContext(ctx, key)

	if err != nil {
		// Handle JWT absence from context...
	}

	fmt.Println(jot)
}

func ExampleFromString() {
	jot, err := jwt.FromString("eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.e30.t-IDcSemACt8x4iTMCda8Yhe3iZaWbvV5XKSTbuAn0M")

	if err != nil {
		// Handle malformed token...
	}

	if err = jot.Verify(jwt.HS256("secret")); err != nil {
		// Handle verification error...
	}

	fmt.Println(jot)
}

func ExampleSign() {
	nextYear := time.Now().Add(24 * 30 * 12 * time.Hour)
	token, err := jwt.Sign(jwt.HS256("secret"), &jwt.Options{ExpirationTime: nextYear})

	if err != nil {
		// ...
	}

	fmt.Println(token)
}
```

## Contribution
### How to help:
- Pull Requests
- Issues
- Opinions

[Go]: https://golang.org
[here]: https://godoc.org/github.com/gbrlsnchs/jwt
