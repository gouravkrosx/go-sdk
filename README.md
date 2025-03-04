# Keploy Go-SDK

[![PkgGoDev](https://pkg.go.dev/badge/github.com/keploy/go-sdk#readme-contents)](https://pkg.go.dev/github.com/keploy/go-sdk#readme-contents)

This is the client SDK for the [Keploy](https://github.com/keploy/keploy) testing platform. You can use this to generate realistic mock files or entire e2e tests for your applications. The **HTTP mocks/stubs and tests are the same format** are are inter-exchangeable.

## Contents

1. [Installation](#installation)
2. [Usage](#usage)
3. [Supported Routers](#supported-routers)
4. [Supported Databases](#supported-databases)
5. [Support Clients](#supported-clients)
6. [Supported JWT Middlewares](#supported-jwt-middlewares)
7. [Mocking/Stubbing for unit tests](#mockingstubbing-for-unit-tests)

## Installation

```bash
go get -u github.com/keploy/go-sdk
```

## Usage

### Create mocks/stubs for your unit-test

These mocks/stubs are realistic and frees you up from writing them manually. Keploy creates `readable/editable` mocks/stubs yaml files which can be referenced in any of your unit-tests tests. An example is mentioned in [Mocking/Stubbing for unit tests](#mockingstubbing-for-unit-tests) section

1. **Wrap your depedency**: To wrap your depdencies, you need to integrate them with the keploy supported wrappers as metioned below. If your dependency is not supported please open a [feature request.](https://github.com/keploy/go-sdk/issues/new?assignees=&labels=&template=feature_request.md&title=)
2. **Record**: To record you can import the keploy mocking library and set the mode to record mode. This should generate a file containing the mocks/stubs.

```go
import(
    "github.com/keploy/go-sdk/keploy"
    "github.com/keploy/go-sdk/mock"
)

// Inside your unit test
...
ctx := mock.NewContext(mock.Config{
	Name: "<name_for_yamls>", // It is unique for every mock/stub. If you dont provide during record it would be generated. Its compulsory during tests.
	Mode: keploy.MODE_RECORD, // It can be MODE_TEST or MODE_OFF. Default is MODE_TEST
	Path: "<local_path_for_yaml>", // optional. It can be relative(./internals) or absolute(/users/xyz/...)
	CTX: <existing context>, // optional. can be used to pass existing running context.
})
...
```

3. **Mock**: To mock dependency as per the content of the generated file (during testing) - just set the `Mode` config to `keploy.MODE_TEST` or remove the variable all together (default is test mode). eg:

```go
ctx := mock.NewContext(mock.Config{
	Name: "<name_for_yamls>", // Should be unique for every mock/stub. Its compulsory during tests.
	Mode: keploy.MODE_TEST,
	Path: "<local_path_for_yaml>", // Optional. It should be the path to the recorded mocks/stubs.It can be relative(./internals) or absolute(/users/xyz/...)
	CTX: <existing context>, // Optional. It can be used to pass existing running context.
})
```

### Generate E2E tests

These tests can be run alongside your manually written `go-tests`and adds coverage to them. This way you can focus on only writing those tests that are hard to e2e test with keploy. Mocks/stubs are also generated and linked to their respective tests. As mentioned above, the tests can also be shared with the clients for mocking (and vice versa!).

1. **Wrap your depedency**: (same as above)
2. **Intialize SDK**: You would need to initialize the keploy SDK

```go
import"github.com/keploy/go-sdk/keploy"

k := keploy.New(keploy.Config{
     App: keploy.AppConfig{
		Name   "<app_name>",         // required field. app_id for grouping testcases.
		Host    "<api_server_host>", // optional. `default:"0.0.0.0"`
		Port    "<api_server_port>", // required.
		Delay   <delay_testing_for>, // optional. `default:"5s"`
		Timeout <context_deadline_for_simulate>, // optional. `default:"60s"`
		Filter  keploy.Filter{
			UrlRegex: "<regular_expression>", // api routes to be tested and recorded
		}, // optional.
		TestPath "", // `optional. default: <absolute-path>/keploy-tests`
		MockPath "", // `optional. default: <absolute-path>/keploy-tests/mocks`
     },
     Server: keploy.ServerConfig{
         URL: "<keploy_host>",        // optional. `default:"https://api.keploy.io"`
         LicenseKey: "<license_key>", // optional. for managed services
     },
    })
```

**Note**: Testcases can be stored on either mongoDB or in yaml files locally. By default, testcases are generated in yaml files locally.
For example:

```go
port := "8080"
 k := keploy.New(keploy.Config{
     App: keploy.AppConfig{
         Name: "my-app",
         Port: port,
     },
     Server: keploy.ServerConfig{
         URL: "http://localhost:6789/api",
     },
 })

```

3. **Record or Test**: You can use the `KEPLOY_MODE` to record or test your application. Eg:

```
export KEPLOY_MODE=keploy.MODE_TEST
```

There are 3 modes:

- **Record**: Sets to record mode.
- **Test**: Sets to test mode.
- **Off**: Turns off all the functionality provided by the API

**Note:** `KEPLOY_MODE` value is case sensitive.

## Supported Routers

### 1. Chi

```go
r := chi.NewRouter()
r.Use(kchi.ChiMiddlewareV5(k))
```

#### Example

```go
import(
  "github.com/keploy/go-sdk/integrations/kchi"
	"github.com/keploy/go-sdk/keploy"
	"github.com/go-chi/chi"
)

func main(){
    r := chi.NewRouter()
    port := "8080"
    k := keploy.New(keploy.Config{
            App: keploy.AppConfig{
                Name: "my_app",
                Port: port,
            },
            Server: keploy.ServerConfig{
                URL: "http://localhost:6789/api",
            },
            })
    r.Use(kchi.ChiMiddlewareV5(k))
    http.ListenAndServe(":" + port, r)
}
```

### 2. Gin

```go
r:=gin.New()
kgin.GinV1(k, r)
```

#### Example

```go
import(
  "github.com/keploy/go-sdk/integrations/kgin/v1"
	"github.com/keploy/go-sdk/keploy"
)

func main(){
	r:=gin.New()
	port := "8080"
	k := keploy.New(keploy.Config{
	  App: keploy.AppConfig{
	      Name: "my_app",
	      Port: port,
	  },
	  Server: keploy.ServerConfig{
	      URL: "http://localhost:6789/api",
	  },
	})
	kgin.GinV1(k, r)
	r.Run(":" + port)
}
```

### 3. Echo

```go
e := echo.New()
e.Use(kecho.EchoMiddlewareV4(k))
```

#### Example

```go
import(
  "github.com/keploy/go-sdk/integrations/kecho/v4"
	"github.com/keploy/go-sdk/keploy"
	"github.com/labstack/echo/v4"
)

func main(){
    e := echo.New()
    port := "8080"
    k := keploy.New(keploy.Config{
      App: keploy.AppConfig{
          Name: "my-app",
          Port: port,
      },
      Server: keploy.ServerConfig{
          URL: "http://localhost:6789/api",
      },
    })
    e.Use(kecho.EchoMiddlewareV4(k))
    e.Start(":" + port)
}
```

### 4. WebGo

#### WebGoV4

```go
router := webgo.NewRouter(cfg, getRoutes())
router.Use(kwebgo.WebgoMiddlewareV4(k))
router.Start()
```

#### WebGoV6

```go
router := webgo.NewRouter(cfg, getRoutes())
router.Use(kwebgo.WebgoMiddlewareV6(k))
router.Start()
```

#### Example

```go
import(
  "github.com/keploy/go-sdk/integrations/kwebgo/v4"
	"github.com/keploy/go-sdk/keploy"
	"github.com/bnkamalesh/webgo/v4"
)

func main(){
    port := "8080"
    k := keploy.New(keploy.Config{
      App: keploy.AppConfig{
          Name: "my-app",
          Port: port,
      },
      Server: keploy.ServerConfig{
          URL: "http://localhost:6789/api",
      },
    })
    router := webgo.NewRouter(&webgo.Config{
		Host:         "",
		Port:         port,
		ReadTimeout:  15 * time.Second,
		WriteTimeout: 60 * time.Second,
	}, []*webgo.Route{})
    router.Use(kwebgo.WebgoMiddlewareV4(k))
    router.Start()
}
```

### 5. Gorilla/Mux

```go
r := mux.NewRouter()
r.Use(kmux.MuxMiddleware(k))
```

#### Example

```go
import(
  "github.com/keploy/go-sdk/integrations/kmux"
	"github.com/keploy/go-sdk/keploy"
	"github.com/gorilla/mux"
  "net/http"
)

func main(){
    r := mux.NewRouter()
    port := "8080"
    k := keploy.New(keploy.Config{
      App: keploy.AppConfig{
          Name: "my-app",
          Port: port,
      },
      Server: keploy.ServerConfig{
          URL: "http://localhost:6789/api",
      },
    })
    r.Use(kmux.MuxMiddleware(k))
    http.ListenAndServe(":"+port, r)
}
```

### 6. FastHttp

Requirement: [valyala/fasthttp](https://github.com/valyala/fasthttp) version >= 1.41.0

```go
mw := kfasthttp.FastHttpMiddleware(k)
```

#### Example

```go
import(
	"github.com/keploy/go-sdk/integrations/kfasthttp"
	"github.com/keploy/go-sdk/keploy"
	"github.com/valyala/fasthttp"
)

func main() {
	k := keploy.New(keploy.Config{
		App: keploy.AppConfig{
			Name: "fasthttp-URL",
			Port: "8080",
		},
		Server: keploy.ServerConfig{
			URL: "http://localhost:6789/api",
		},
	})

	mw := kfasthttp.FastHttpMiddleware(k)
	m := func(ctx *fasthttp.RequestCtx) {
		switch string(ctx.Path()) {
		case "/index":
			index(ctx)
		default:
			ctx.Error("not found", fasthttp.StatusNotFound)
		}
	}
	log.Fatal(fasthttp.ListenAndServe(":8080", mw(m)))
}
```

### 7. net/http (core)

```go
handler := khttp.KMiddleware(handler, k)
```

#### Example

```go
import(

	"github.com/keploy/go-sdk/integrations/khttp"
	"github.com/keploy/go-sdk/keploy"
)

func main(){

   	var handler http.Handler = &myHandler{}
	port := "8080"
	k := keploy.New(keploy.Config{
		App: keploy.AppConfig{
			Name: "my_app",
			Port: port,
		},
		Server: keploy.ServerConfig{
			URL: "http://localhost:6789/api",
		},
	})

	handler = khttp.KMiddleware(handler, k)

	http.Handle("/hello", handler)

	err := http.ListenAndServe(":8080", handler)
	if err != nil {
		panic(err)
	}
}
```

#### Example with goswagger

```go
import(
	"github.com/keploy/go-sdk/integrations/khttp"
	"github.com/keploy/go-sdk/keploy"
)
	//This function is used to add middlewares in your configurable file
	func setupMiddlewares(handler http.Handler) http.Handler {
	port := "8080"
	k := keploy.New(keploy.Config{
		App: keploy.AppConfig{
			Name: "my_app",
			Port: port,
		},
		Server: keploy.ServerConfig{
			URL: "http://localhost:6789/api",
		},
	})
	return khttp.KMiddleware(handler, k)
}
```

### 8. gRPC Server

Testcases can be generated for gRPC unary methods by just registering keploy's gRPC Unary interceptor(from go-sdk/integrations/kgrpcserver package) in grpc.NewServer and enabling reflection for server.

#### Example

```go
import(
	"google.golang.org/grpc"
	"google.golang.org/grpc/reflection"
	"github.com/keploy/go-sdk/integrations/kgrpcserver"
	"github.com/keploy/go-sdk/keploy"
)

func main() {
	// port on which gRPC server will run
	port := "8080"
	k := keploy.New(keploy.Config{
	  App: keploy.AppConfig{
		  Name: "my-app",
		  Port: port,
	  },
	  Server: keploy.ServerConfig{
		  URL: "http://localhost:6789/api",
	  },
	})

	// make new gRPC server with keploy unary interceptor
	srv := grpc.NewServer(kgrpcserver.UnaryInterceptor(k))
	reflection.Register(srv)
}
```

## Supported Databases

### 1. MongoDB

```go
import("github.com/keploy/go-sdk/integrations/kmongo")

db  := client.Database("testDB")
col := kmongo.NewCollection(db.Collection("Demo-Collection"))
```

Following operations are supported:<br>

- FindOne - Err and Decode method of mongo.SingleResult<br>
- Find - Next, TryNext, Err, Close, All and Decode methods of mongo.cursor<br>
- InsertOne<br>
- InsertMany<br>
- UpdateOne<br>
- UpdateMany<br>
- DeleteOne<br>
- DeleteMany<br>
- CountDocuments<br>
- Distinct<br>
- Aggregate - Next, TryNext, Err, Close, All and Decode methods of mongo.cursor

### 2. DynamoDB

```go
import("github.com/keploy/go-sdk/integrations/kddb")

client := kddb.NewDynamoDB(dynamodb.New(sess))
```

Following operations are supported:<br>

- QueryWithContext
- GetItemWithContext
- PutItemWithContext

### 3. SQL Driver

Keploy inplements most of the sql driver's interface for mocking the outputs of sql queries which are called from your API handler.

Since, keploy uses request context for mocking outputs of SQL queries thus, SQL methods having request context as parameter should be called from API handler.

#### v1

This version records the outputs and store them as binary in exported yaml files

#### v2

This version records and stores the outputs as readable/editable format in exported yaml file.
Sample:

```yaml
version: api.keploy.io/v1beta1
kind: SQL
name: Sample-App # App_Id from keploy config or mock name from mock.Config
spec:
  metadata:
    name: SQL
    operation: QueryContext.Close
    type: SQL_DB
  type: table
  table:
    cols:
      - name: id
        type: int64
        precision: 0
        scale: 0
      - name: uuid
        type: "[]uint8"
        precision: 0
        scale: 0
      - name: name
        type: string
        precision: 0
        scale: 0
    rows:
      - "[`3` | `[50 101 101]` | `qwertt2` | ]"
  int: 0
  error:
    - nil
    - nil
```

Here is an example for postgres driver and binary encoded outputs -

```go
    import (
        "github.com/keploy/go-sdk/integrations/ksql/v1" // the outputs of sql queries are stored as binary encoded in exported yaml files
        "github.com/lib/pq"
    )
    func main(){
        // Register keploy sql driver to database/sql package.
        driver := ksql.Driver{Driver: pq.Driver{}}
		sql.Register("keploy", &driver)

        pSQL_URI := fmt.Sprintf("host=%s user=%s dbname=%s sslmode=disable password=%s port=%s", "localhost", "postgres", "Book_Keeper", "8789", "5432")
        // keploy driver will internally open the connection using dataSourceName string parameter
        db, err := sql.Open("keploy", pSQL_URI)
        if err!=nil{
            log.Fatal(err)
        } else {
            fmt.Println("Successfully connected to postgres")
        }
        defer db.Close

        r:=gin.New()
        kgin.GinV1(kApp, r)
        r.GET("/gin/:color/*type", func(c *gin.Context) {
            // ctx parameter of PingContext should be request context.
            err = db.PingContext(r.Context())
            if err!=nil{
                log.Fatal(err)
            }
            id := 47
            result, err := db.ExecContext(r.Context(), "UPDATE balances SET balance = balance + 10 WHERE user_id = ?", id)
            if err != nil {
                log.Fatal(err)
            }
        }))
    }
```

**Note**: Its compatible with gORM. To integerate with gORM set DisableAutomaticPing of gorm.Config to true. Also pass request context to methods as params.
Example for gORM with GCP-Postgres driver:

```go
    import (
		gcppostgres "github.com/GoogleCloudPlatform/cloudsql-proxy/proxy/dialers/postgres"
        "github.com/keploy/go-sdk/integrations/ksql/v1" // the outputs of sql queries are stored as binary encoded in exported yaml files
        "gorm.io/driver/postgres"
	    "gorm.io/gorm"
    )
    type Person struct {
        gorm.Model
        Name  string
        Email string `gorm:"typevarchar(100);unique_index"`
        Books []Book
    }
    type Book struct {
        gorm.Model
        Title      string
        Author     string
        CallNumber int64 `gorm:"unique_index"`
        PersonID   int
    }
    func main(){
        // Register keploy sql driver to database/sql package.
        driver := ksql.Driver{Driver: gcppostgres.Driver{}}
        sql.Register("keploy", &driver)

        pSQL_URI := fmt.Sprintf("host=%s user=%s dbname=%s sslmode=disable password=%s", GCPHost, "postgres", "Book_Keeper", "8789", "5432")

        // set DisableAutomaticPing to true so that .
        pSQL_DB, err :=  gorm.Open( postgres.New(postgres.Config{
                DriverName: "keploy",
                DSN: pSQL_URI
            }), &gorm.Config{
                DisableAutomaticPing: true
        }
        pSQL_DB.AutoMigrate(&Person{})
        pSQL_DB.AutoMigrate(&Book{})
        r:=gin.New()
        kgin.GinV1(kApp, r)
        r.GET("/gin/:color/*type", func(c *gin.Context) {
            // set the context of *gorm.DB with request's context of http Handler function before queries.
            pSQL_DB = pSQL_DB.WithContext(c.Request.Context())
            // Find
            var (
                people []Book
            )
            x := pSQL_DB.Find(&people)
        }))
    }
```

### 4. Elasticsearch

The elastic-search client uses http client to do CRUD operations. There is a Transport field in _elasticsearch.config_ which allows you to completely replace the default HTTP client used by the package.So, we use _khttp_ as an interceptor and assign it to the Transport field.
Here is an example of making elastic search client with keploy's http interceptor -

```go
import (
	"net/http"
	"github.com/elastic/go-elasticsearch/v8"
	"github.com/keploy/go-sdk/integrations/khttpclient"
)

func ConnectWithElasticsearch(ctx context.Context) *elasticsearch.Client {
	// integrate http with keploy
	interceptor := khttpclient.NewInterceptor(http.DefaultTransport)
	newClient, err := elasticsearch.NewClient(elasticsearch.Config{
		Addresses: []string{
			"http://localhost:9200",
		},
		// use khttp as custom http client
		Transport: interceptor,
	})
	if err != nil {
		panic(err)
	}
	return newClient
}
```

### 5. Redis

```go
import(
	"context"
	"time"
	"github.com/go-redis/redis/v8"
	"github.com/keploy/go-sdk/integrations/kredis"
)

type redisCache struct {
	host    string
	db      int
	expires time.Duration
}

func (cache *redisCache) getClient() redis.UniversalClient {
	client := redis.NewClient(&redis.Options{
		Addr:     cache.host,
		Password: "",
		DB:       cache.db,
	})
	return kredis.NewRedisClient(client)
}
```

Following operations are supported:<br>

- Get
- Set
- Del

## Supported Clients

### net/http

```go
interceptor := khttpclient.NewInterceptor(http.DefaultTransport)
client := http.Client{
    Transport: interceptor,
}
```

#### Example

```go
import("github.com/keploy/go-sdk/integrations/khttpclient")

func main(){
	// initialize a gorilla mux
	r := mux.NewRouter()
	// keploy config
	port := "8080"
	kApp := keploy.New(keploy.Config{
		App: keploy.AppConfig{
			Name: "Mux-Demo-app",
			Port: port,
		},
		Server: keploy.ServerConfig{
			URL: "http://localhost:6789/api",
		},
	})
	// configure mux for integeration with keploy
	kmux.Mux(kApp, r)
	// configure http client with keploy's interceptor
	interceptor := khttpclient.NewInterceptor(http.DefaultTransport)
	client := http.Client{
		Transport: interceptor,
	}

	r.HandleFunc("/mux/httpDo", func(w http.ResponseWriter, r *http.Request){
		putBody, _ := json.Marshal(map[string]interface{}{
		    "name":  "Ash",
		    "age": 21,
		    "city": "Palet town",
		})
		PutBody := bytes.NewBuffer(putBody)
		// Use handler request's context or SetContext before http.Client.Do method call
		req,err := http.NewRequestWithContext(r.Context(), http.MethodPut, "https://example.com/updateDocs", PutBody)
		req.Header.Set("Content-Type", "application/json; charset=utf-8")
		if err!=nil{
		    log.Fatal(err)
		}
		resp,err := cl.Do(req)
		if err!=nil{
		    log.Fatal(err)
		}
		defer resp.Body.Close()
		body, err := io.ReadAll(resp.Body)
		if err!=nil{
		    log.Fatal(err)
		}
		fmt.Println(" response Body: ", string(body))

	})
	
	r.HandleFunc("/mux/httpGet",func (w http.ResponseWriter, r *http.Request)  {
		// ONLY USE THIS IF YOU CANT ADD CONTEXT TO YOUR REQUEST OBJECT AS ABOVE. 
		// THIS IS NOT CONCURRENT SAFE BECAUSE WE ARE SETTING GLOBAL CONTEXT.  
		// SetContext should always be called once in a http handler before http.Client's Get or Post or Head or PostForm method.
        	// Passing requests context as parameter.
		interceptor.SetContext(r.Context())
		// make Get, Post, etc request to external http service
		resp, err := client.Get("https://example.com/getDocs")
		if err != nil {
			log.Fatal(err)
		}
		defer resp.Body.Close()
		body, err := io.ReadAll(resp.Body)
		fmt.Println("BODY : ", body)
	})

	// gcp compute API integeration
	client, err := google.DefaultClient(context.TODO(), compute.ComputeScope)
	if err != nil {
		fmt.Println(err)
	}
	// add keploy interceptor to gcp httpClient
	intercept := khttpclient.NewInterceptor(client.Transport)
	client.Transport = intercept

	r.HandleFunc("/mux/gcpDo", func(w http.ResponseWriter, r *http.Request){
		computeService, err := compute.NewService(r.Context(), option.WithHTTPClient(client), option.WithCredentialsFile("/Users/abc/auth.json"))
		zoneListCall := computeService.Zones.List(project)
		zoneList, err := zoneListCall.Do()
	})
}
```

**Note**: ensure to pass request context to all external requests like http requests, db calls, etc.

### gRPC Client

The outputs of external gRPC calls from API handlers can be mocked by registering keploy's gRPC client interceptor(called WithClientUnaryInterceptor of go-sdk/integrations/kgrpc package).

```go
conn, err := grpc.Dial(address, grpc.WithInsecure(), kgrpc.WithClientUnaryInterceptor(k))
```

#### Example

```go
import(
	"github.com/keploy/go-sdk/integrations/kgrpc"
	"github.com/keploy/go-sdk/keploy"
)

func main() {
	port := "8080"
	k := keploy.New(keploy.Config{
	  App: keploy.AppConfig{
		  Name: "my-app",
		  Port: port,
	  },
	  Server: keploy.ServerConfig{
		  URL: "http://localhost:6789/api",
	  },
	})

	// Make gRPC client connection
	conn, err := grpc.Dial(address, grpc.WithInsecure(), kgrpc.WithClientUnaryInterceptor(k))
}
```

**Note**: Currently streaming is not yet supported.

## Supported JWT Middlewares

### jwtauth

Middlewares which can be used to authenticate. It is compatible for Chi, Gin and Echo router. Usage is similar to go-chi/jwtauth. Adds ValidationOption to mock time in test mode.

#### Example

```go
package main

import (
	"github.com/gin-gonic/gin"
	"github.com/go-chi/chi"
	"github.com/labstack/echo/v4"

	"github.com/benbjohnson/clock"
	"github.com/keploy/go-sdk/integrations/kchi"
	"github.com/keploy/go-sdk/integrations/kecho/v4"
	"github.com/keploy/go-sdk/integrations/kgin/v1"

	"github.com/keploy/go-sdk/integrations/kjwtauth"
	"github.com/keploy/go-sdk/keploy"
)

var (
	kApp      *keploy.Keploy
	tokenAuth *kjwtauth.JWTAuth
)

func init() {
        // Initialize kaploy instance
	port := "6060"
	kApp = keploy.New(keploy.Config{
		App: keploy.AppConfig{
			Name: "client-echo-App",
			Port: port,
		},
		Server: keploy.ServerConfig{
			URL: "http://localhost:6789/api",
		},
	})
        // Generate a JWTConfig
	tokenAuth = kjwtauth.New("HS256", []byte("mysecret"), nil, kApp)

	claims := map[string]interface{}{"user_id": 123}
	kjwtauth.SetExpiryIn(claims, 20*time.Second)
        // Create a token string
	_, tokenString, _ := tokenAuth.Encode(claims)
	fmt.Printf("DEBUG: a sample jwt is %s\n\n", tokenString)
}

func main() {
	addr := ":6060"

	fmt.Printf("Starting server on %v\n", addr)
	http.ListenAndServe(addr, echoRouter())
}

func chiRouter()  http.Handler {
        // Chi example(comment echo, gin to use chi)
	r := chi.NewRouter()
	kchi.ChiV5(kApp, r)
        // Protected routes
	r.Group(func(r chi.Router) {
		// Seek, verify and validate JWT tokens
		r.Use(kjwtauth.VerifierChi(tokenAuth))

		// Handle valid / invalid tokens. In this example, we use
		// the provided authenticator middleware, but you can write your
		// own very easily, look at the Authenticator method in jwtauth.go
		// and tweak it, its not scary.
		r.Use(kjwtauth.AuthenticatorChi)

		r.Get("/admin", func(w http.ResponseWriter, r *http.Request) {
			_, claims, _ := kjwtauth.FromContext(r.Context())
			fmt.Println("requested admin")
			w.Write([]byte(fmt.Sprintf("protected area, Hi %v", claims["user_id"])))
		})
	})
	// Public routes
    	r.Get("/", func(w http.ResponseWriter, r *http.Request) {
        	w.Write([]byte("welcome"))
    	})

	return r
}

func echoRouter() http.Handler {
        // Echo example
	er := echo.New()
        // add keploy's echo middleware
	kecho.EchoV4(kApp, er)
        // Public route
	er.GET("/", func(c echo.Context) error {
		return c.String(http.StatusOK, "Accessible")
	})
        // Protected route
	er.GET("echoAdmin", func(c echo.Context) error {
		_, claims, _ := kjwtauth.FromContext(c.Request().Context())
		fmt.Println("requested admin")
		return c.String(http.StatusOK, fmt.Sprint("protected area, Hi fin user: %v", claims["user_id"]))
	}, kjwtauth.VerifierEcho(tokenAuth), kjwtauth.AuthenticatorEcho)
	return er
}

func ginRouter() http.Handler {
        // Gin example(comment echo example to use gin)
	gr := gin.New()
	kgin.GinV1(kApp, gr)
        // Public route
	gr.GET("/", func(ctx *gin.Context) {
		ctx.Writer.Write([]byte("welcome to gin"))
	})
        // Protected route
	auth := gr.Group("/auth")
	auth.Use(kjwtauth.VerifierGin(tokenAuth))
	auth.Use(kjwtauth.AuthenticatorGin)
	auth.GET("/ginAdmin", func(c *gin.Context) {
		_, claims, _ := kjwtauth.FromContext(c.Request.Context())
		fmt.Println("requested admin")
		c.Writer.Write([]byte(fmt.Sprintf("protected area, Hi fin user: %v", claims["user_id"])))
	})
	return gr
}
```

## Mocking/Stubbing for unit tests

Mocks/Stubs can be generated for external dependency calls of go unit tests as `readable/editable` yaml files using Keploy.

### Example

```go
import (
	"bytes"; "context"; "io"; "net/http"; "testing"

	"github.com/go-test/deep"
	"github.com/keploy/go-sdk/integrations/khttpclient"
	"github.com/keploy/go-sdk/keploy"
	"github.com/keploy/go-sdk/mock"
)

func TestExample(t *testing.T) {
	for _, tt := range []struct {
		input struct {
			tcsName string
			reqURL  string
		}
		output struct {
			resp string
			err  error
		}
	}{
		{
			input: struct {
				tcsName string
				reqURL  string
			}{
				tcsName: "sample-mock",
				reqURL:  "https://api.keploy.io/healthz",
			},
			output: struct {
				resp string
				err  error
			}{
				resp: "ok",
				err:  nil,
			},
		},
	} {
		ctx := context.Background()
		// Integration for keploy unit-tests mocks/stubs.
		ctx = mock.NewContext(mock.Config{
			Mode:      keploy.MODE_RECORD,   // Keploy mode on which unit test will run. Possible values: MODE_TEST or MODE_RECORD.
			Name:      tt.input.tcsName,     // Unique names for testcases of a unit test. If it is empty then, a unique id is generated by keploy during RECORD.
			CTX:       ctx,                  // Context in which KeployContext will be stored. If it is nil, then KeployContext is stored in context.Background.
			Path:      "./",                 // Path in which Keploy "/mocks" will be generated. Default: current working directroy.
			OverWrite: false,                // OverWrite is used to compare new outputs of external calls with previously recorded outputs during record.
			Remove:    []string{},           // removes given http fields from yaml. Format: "<req_OR_resp_OR_all>.<header_OR_body>.<FIELD_NAME>"
			Replace:   map[string]string{},  // replaces values of given http fields. Format: key (can be "header.<KEY>", "domain", "method", "proto_major", "proto_minor")
		})

		// Integration for Keploy supported httpClient
		interceptor := khttpclient.NewInterceptor(http.DefaultTransport)
		client := http.Client{
			Transport: interceptor,
		}
		// Http request with generated context
		req, err := http.NewRequestWithContext(ctx, "GET", tt.input.reqURL, bytes.NewBuffer([]byte{}))
		if err != nil {
			t.Error("failed to make http request", err)
		}
		// Make the http call
		resp, actErr := client.Do(req)
		if deep.Equal(tt.output.err, actErr) != nil {
			t.Fatal("Testcase Failed. Expected Error:", tt.output.err, " Actual Error: ", actErr)
		}
		defer resp.Body.Close()
		bodyBytes, err := io.ReadAll(resp.Body)
		if err != nil {
			t.Error("failed to read response body", err)
		}
		actRespBody := string(bodyBytes)
		if deep.Equal(tt.output.resp, actRespBody) != nil {
			t.Fatal("Testcase Failed. Expected Response body:", tt.output.resp, " Actual Response body: ", actRespBody)
		}
	}
}

```
