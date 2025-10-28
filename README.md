# go-localstack

[![Test](https://github.com/RyanW8/go-localstack/workflows/Test/badge.svg)](https://github.com/RyanW8/go-localstack/actions/workflows/test.yml)
[![codecov](https://codecov.io/gh/RyanW8/go-localstack/branch/main/graph/badge.svg)](https://codecov.io/gh/RyanW8/go-localstack)
[![CodeQL](https://github.com/RyanW8/go-localstack/workflows/CodeQL/badge.svg)](https://github.com/RyanW8/go-localstack/actions/workflows/codeql-analysis.yml)
[![Go Report Card](https://goreportcard.com/badge/github.com/RyanW8/go-localstack)](https://goreportcard.com/report/github.com/RyanW8/go-localstack)
[![PkgGoDev](https://pkg.go.dev/badge/github.com/RyanW8/go-localstack)](https://pkg.go.dev/github.com/RyanW8/go-localstack)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://github.com/gojp/goreportcard/blob/master/LICENSE)

Go Wrapper for using [localstack](https://github.com/localstack/localstack) in go testing

# Installation

Please make sure that you have Docker installed.

```bash
go get github.com/RyanW8/go-localstack
```

# Usage

With SDK V2 (using EndpointResolverV2).
Please have a look at [resolvers](resolver.go) for a complete list of resolvers.
```go
func ExampleLocalstackSdkV2EndpointResolverV2(t *testing.T) {
    l, err := localstack.NewInstance()
    if err != nil {
        t.Fatalf("Could not connect to Docker %v", err)
    }
    if err := l.Start(); err != nil {
        t.Fatalf("Could not start localstack %v", err)
    }
    t.Cleanup(func() {
        if err := l.Stop(); err != nil {
            t.Fatalf("Could not stop localstack %v", err)
        }
    })
    
    cfg, err := config.LoadDefaultConfig(ctx,
        config.WithRegion("us-east-1"),
        config.WithCredentialsProvider(credentials.NewStaticCredentialsProvider("dummy", "dummy", "dummy")),
    )
    if err != nil {
        t.Fatalf("Could not get config %v", err)
    }
    resolver := localstack.NewDynamoDbResolverV2(i)
    client := dynamodb.NewFromConfig(cfg, dynamodb.WithEndpointResolverV2(resolver))
	
    myTestWithV2Client(client)
}
```

With SDK V2 (using EndpointResolverV1)
```go
func ExampleLocalstackSdkV2(t *testing.T) {
    l, err := localstack.NewInstance()
    if err != nil {
        t.Fatalf("Could not connect to Docker %v", err)
    }
    if err := l.Start(); err != nil {
        t.Fatalf("Could not start localstack %v", err)
    }
    t.Cleanup(func() {
        if err := l.Stop(); err != nil {
            t.Fatalf("Could not stop localstack %v", err)
        }
	})
    
    cfg, err := config.LoadDefaultConfig(ctx,
        config.WithRegion("us-east-1"),
        config.WithEndpointResolverWithOptions(aws.EndpointResolverWithOptionsFunc(func(_, _ string, _ ...interface{}) (aws.Endpoint, error) {
            return aws.Endpoint{
			    PartitionID:       "aws", 
			    URL:               l.EndpointV2(localstack.SQS), 
			    SigningRegion:     "us-east-1", 
			    HostnameImmutable: true,
		    }, nil
        })),
        config.WithCredentialsProvider(credentials.NewStaticCredentialsProvider("dummy", "dummy", "dummy")),
    )
    if err != nil {
        t.Fatalf("Could not get config %v", err)
    }
    
    myTestWithV2(cfg)
}
```

With SDK V1
```go
func TestWithLocalStack(t *testing.T) {
    l, err := localstack.NewInstance()
    if err != nil {
        t.Fatalf("Could not connect to Docker %v", err)
    }
    if err := l.Start(); err != nil {
        t.Fatalf("Could not start localstack %v", err)
    }
    t.Cleanup(func() {
        if err := l.Stop(); err != nil {
            t.Fatalf("Could not stop localstack %v", err)
        }
    })

    myTestWith(&aws.Config{
        Credentials: credentials.NewStaticCredentials("not", "empty", ""),
        DisableSSL:  aws.Bool(true),
        Region:      aws.String(endpoints.UsWest1RegionID),
        Endpoint:    aws.String(l.Endpoint(localstack.SQS)),
    })
}
```

