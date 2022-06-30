# SIWE_GO: Edge cases failing

Ok so at [Tally](https://tally.xyz) we use Golang in the backend to do some tasks. And part of the signin is handled by it.
But the code of the library : [siwe_go](github.com/spruceid/siwe_go)

## Replicate the issue.

Ok so a ledger signin is failing in siwe-go but not in siwe-js, to replicate the bug please, follow the next steps:

1. Clone this repository.

	`git clone https://github.com/afa7789/siwe_go.git siwe_go_tally && cd siwe_go_tally`

2. Recursively clone sub repositories. 

	`git submodule update --init --recursive`

3. Copy both of this cases to the test files:

Run the patch files to make the changes needed in the test files:

```bash
	patch siwe-js/test/parsing_positive.json -i patches/parsing_positive.patch
	patch siwe-js/test/verification_positive.json -i patches/verification_positive.patch
```

Or you can copy and paste from here:

_siwe-js/test/parsing_positive.json_
```json
    "tally_example": {
        "message": "www.tally.xyz wants you to sign in with your Ethereum account:\n0xc95EB884FE852e241D409234bfC7045CB9E31BD7\n\nSign in with Ethereum to Tally\n\nURI: https://tally.xyz\nVersion: 1\nChain ID: 1\nNonce: 15050747\nIssued At: 2022-06-30T14:08:51.382Z",
        "fields": {
            "domain": "www.tally.xyz",
            "address": "0xc95EB884FE852e241D409234bfC7045CB9E31BD7",
            "statement": "Sign in with Ethereum to Tally",
            "uri": "https://tally.xyz",
            "version": "1",
            "chainId": 1,
            "nonce": "15050747",
            "issuedAt": "2022-06-30T14:08:51.382Z"
        }
    }
```

_siwe-js/test/verification_positive.json_
```json
	"tally_example": {
		"domain": "www.tally.xyz",
		"address": "0xc95EB884FE852e241D409234bfC7045CB9E31BD7",
		"statement": "Sign in with Ethereum to Tally",
		"uri": "https://tally.xyz",
		"version": "1",
		"chainId": 1,
		"nonce": "15050747",
		"issuedAt": "2022-06-30T14:08:51.382Z",
		"signature": "0x8c46b6eb8505939892d8e9b075f89f8277321b17b993151f37810cdda38cce6f4a85909d2b53e6a14629c74c0ac38bf4becde78ee5b2529812bf6cceaf7b2a2501"
	}
```

4. Install everything in the js package just to be sure it will work.

```bash
	pushd siwe-js
	npm install
	npm install --dev # not sure if needed.
	pushd packages/siwe
	npm install
	npm install --dev # not sure if needed.
	popd
	pushd packages/siwe-parser
	npm install
	npm install --dev # not sure if needed.
	popd
	popd
```

5. Run the tests in siwe_js and see that it works.

```bash
	pushd siwe-js
	npx jest
	popd
```

Run then more separated to see the tally example being printed as ok.

```bash
	pushd siwe-js/packages/siwe-parser
	npx jest
	popd
```

```bash
	pushd siwe-js/packages/siwe
	npx jest
	popd
```

Results:

```bash
Round Trip
	✓ Generates a Successfully Verifying message: tally_example (367 ms)
###
Message Generation
    ✓ Generates message successfully: tally_example (20 ms)
###
Message verification
	✓ Verificates message successfully: tally_example (288 ms)
###
Successfully parses with RegExp Client
    ✓ Parses message successfully: tally_example (8 ms)
###
Successfully parses with ABNF Client
    ✓ Parses message successfully: tally_example (216 ms)
```


6. Run the tests in siwe_go and see that it fails.

```bash
	go test -v .
	# or more specifically:
	go test -v -timeout 30s -run ^TestGlobalTestVector$

```

Error output:

The error we are getting is `Invalid signature recovery byte`

```bash
=== RUN   TestGlobalTestVector
    siwe_test.go:378: 
                Error Trace:    siwe_test.go:378
                                                        siwe_test.go:411
                Error:          Expected nil, but got: &siwe.InvalidSignature{string:"Invalid signature recovery byte"}
                Test:           TestGlobalTestVector
                Messages:       tally_example
--- FAIL: TestGlobalTestVector (0.00s)
FAIL
exit status 1
FAIL    github.com/spruceid/siwe-go     0.020s
```