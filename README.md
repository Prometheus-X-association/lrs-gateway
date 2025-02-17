# Personal Learning Record Store BB
The Personal Learning Record Store (PLRS) service is a cloud-based platform that empowers individuals to store and manage their learning records centrally. Unlike traditional Learning Record Stores (LRS) owned by training organizations, a PLRS is directly owned by the learner, providing unparalleled control over their personal learning data. With PLRS, users can easily access, download, and reuse their learning records, enhancing data portability and GDPR compliance. This service not only tracks learning activities, achievements, and progress but also facilitates secure and private sharing of this information when desired. Embrace the future of learning with a PLRS—your personal cloud for lifelong education.

## Design Document
See the design document [here](docs/design-document.md).

## Building instructions

To build the project, run:

```bash
docker compose build 
```

## Running instructions


To bootstrap a test environment on your machine, clone this project first and
run the `bootstrap` Makefile target:

```bash
make bootstrap
```

This command will create required `.env` file (you may want to edit it for your
test environment), build the Ralph's Docker image and start a single node
Elasticsearch cluster _via_ Docker compose.

You can check the `elasticsearch` service status using the `status` helper:

```bash
make status # This is an alias for: docker compose ps
```

You may now start the LRS server using:

```bash
make run
```

The server should be up and running at
[http://localhost:8100](http://localhost:8100). You can check its status using
the heartbeat probe:

```bash
curl http://localhost:8100/__heartbeat__
```

The expected answer should be:

```json
{"database":"ok"}
```

If the database status is satisfying, you are now ready to send xAPI statements
to the LRS:

```bash
gunzip -c data/statements.json.gz | \
head -n 100 | \
jq -s . | \
curl -Lk \
    --user ralph:secret \
    -X POST \
    -H "Content-Type: application/json" \
    -d @- \
    http://localhost:8100/xAPI/statements/
```

The command above fetches one hundred (100) example xAPI statements and sends them to the LRS
using `curl`.

You can get them back from the LRS using `curl` to query the
`/xAPI/statements/` endpoint:

```bash
curl -s \
    --user ralph:secret \
    -H "Content-Type: application/json" \
    http://localhost:8100/xAPI/statements/ \ |
jq
```

> Note that using `jq` is optional in this case, it is used to improve response
> readability. It is not required to install it to run this snippet.


## Example usage

### POST single statement

#### Endpoint

`POST /xAPI/statements/`

#### Example input

```json
{
    "id": "12345678-1234-5678-1234-567812345678",
    "actor":{
        "mbox":"mailto:xapi@adlnet.gov"
    },
    "verb":{
        "id":"http://adlnet.gov/expapi/verbs/created",
        "display":{
            "en-US":"created"
        }
    },
    "object":{
        "id":"http://example.adlnet.gov/xapi/example/activity"
    },
    "timestamp": "2024-12-31T12:00:00.000000"
}
```

#### Expected output

HTTP response status code: `200 OK`

```json
["12345678-1234-5678-1234-567812345678"]
```

### POST multiple statements

#### Endpoint

`POST /xAPI/statements/`

#### Example input

```json
[
    {
        "id": "12345678-1234-5678-1234-567812345678",
        "actor":{
            "mbox":"mailto:xapi@adlnet.gov"
        },
        "verb":{
            "id":"http://adlnet.gov/expapi/verbs/created",
            "display":{
                "en-US":"created"
            }
        },
        "object":{
            "id":"http://example.adlnet.gov/xapi/example/activity"
        },
        "timestamp": "2024-12-31T12:00:00.000000"
    },
    {
        "id": "876543218765-4321-8765-4321-87654321",
        "actor":{
            "mbox":"mailto:xapi@adlnet.gov"
        },
        "verb":{
            "id":"http://adlnet.gov/expapi/verbs/attended",
            "display":{
                "en-US":"attended"
            }
        },
        "object":{
            "id":"http://example.adlnet.gov/xapi/example/meeting"
        },
        "timestamp": "2025-01-01T12:00:00.000000"
    },

]
```

#### Expected output

HTTP response status code: `200 OK`

```json
["12345678-1234-5678-1234-567812345678", "876543218765-4321-8765-4321-87654321"]
```

### PUT single statement

#### Endpoint

`PUT /xAPI/statements/`

#### Example input

```json
{
    "id": "12345678-1234-5678-1234-567812345678",
    "actor":{
        "mbox":"mailto:xapi@adlnet.gov"
    },
    "verb":{
        "id":"http://adlnet.gov/expapi/verbs/created",
        "display":{
            "en-US":"created"
        }
    },
    "object":{
        "id":"http://example.adlnet.gov/xapi/example/activity"
    },
    "timestamp": "2024-12-31T12:00:00.000000"
}
```

#### Expected output

HTTP response status code: `204 No Content`

```json
["12345678-1234-5678-1234-567812345678"]
```

### GET all statements

#### Endpoint

`GET /xAPI/statements/`

#### Expected output

```json
[
    {
        "id": "12345678-1234-5678-1234-567812345678",
        "actor":{
            "mbox":"mailto:xapi@adlnet.gov"
        },
        "verb":{
            "id":"http://adlnet.gov/expapi/verbs/created",
            "display":{
                "en-US":"created"
            }
        },
        "object":{
            "id":"http://example.adlnet.gov/xapi/example/activity"
        },
        "timestamp": "2024-12-31T12:00:00.000000"
    },
    {
        "id": "876543218765-4321-8765-4321-87654321",
        "actor":{
            "mbox":"mailto:xapi@adlnet.gov"
        },
        "verb":{
            "id":"http://adlnet.gov/expapi/verbs/attended",
            "display":{
                "en-US":"attended"
            }
        },
        "object":{
            "id":"http://example.adlnet.gov/xapi/example/meeting"
        },
        "timestamp": "2025-01-01T12:00:00.000000"
    },

]
```

### GET statements by id

#### Endpoint

`GET /xAPI/statements/?statementId=12345678-1234-5678-1234-567812345678`

#### Expected output

```json
{
    "id": "12345678-1234-5678-1234-567812345678",
    "actor":{
        "mbox":"mailto:xapi@adlnet.gov"
    },
    "verb":{
        "id":"http://adlnet.gov/expapi/verbs/created",
        "display":{
            "en-US":"created"
        }
    },
    "object":{
        "id":"http://example.adlnet.gov/xapi/example/activity"
    },
    "timestamp": "2024-12-31T12:00:00.000000"
}
```

### GET statements by verb

#### Endpoint

`GET /xAPI/statements/?verb=http%3A%2F%2Fadlnet.gov%2Fexpapi%2Fverbs%2Fcreated`

#### Expected output

```json
{
    "id": "12345678-1234-5678-1234-567812345678",
    "actor":{
        "mbox":"mailto:xapi@adlnet.gov"
    },
    "verb":{
        "id":"http://adlnet.gov/expapi/verbs/created",
        "display":{
            "en-US":"created"
        }
    },
    "object":{
        "id":"http://example.adlnet.gov/xapi/example/activity"
    },
    "timestamp": "2024-12-31T12:00:00.000000"
}
```

### GET statements by activity

#### Endpoint

`GET /xAPI/statements/?activity=http%3A%2F%2Fexample.adlnet.gov%2Fxapi%2Fexample%2Fmeeting`

#### Expected output

```json
{
    "id": "876543218765-4321-8765-4321-87654321",
    "actor":{
        "mbox":"mailto:xapi@adlnet.gov"
    },
    "verb":{
        "id":"http://adlnet.gov/expapi/verbs/attended",
        "display":{
            "en-US":"attended"
        }
    },
    "object":{
        "id":"http://example.adlnet.gov/xapi/example/meeting"
    },
    "timestamp": "2025-01-01T12:00:00.000000"
}
```

### GET statements since

#### Endpoint

`GET /xAPI/statements/?since=2025-01-01T09:00:00.000000`

#### Expected output

```json
{
    "id": "876543218765-4321-8765-4321-87654321",
    "actor":{
        "mbox":"mailto:xapi@adlnet.gov"
    },
    "verb":{
        "id":"http://adlnet.gov/expapi/verbs/attended",
        "display":{
            "en-US":"attended"
        }
    },
    "object":{
        "id":"http://example.adlnet.gov/xapi/example/meeting"
    },
    "timestamp": "2025-01-01T12:00:00.000000"
}
```

### GET statements until

#### Endpoint

`GET /xAPI/statements/?until=2025-01-01T09:00:00.000000`

#### Expected output

```json
{
    "id": "12345678-1234-5678-1234-567812345678",
    "actor":{
        "mbox":"mailto:xapi@adlnet.gov"
    },
    "verb":{
        "id":"http://adlnet.gov/expapi/verbs/created",
        "display":{
            "en-US":"created"
        }
    },
    "object":{
        "id":"http://example.adlnet.gov/xapi/example/activity"
    },
    "timestamp": "2024-12-31T12:00:00.000000"
}
```

## Unit testing
### Setup test environment
### Run tests
### Expected results

## Component-level testing
### Setup test environment
### Run tests
### Expected results

## License

This work is released under the MIT License (see [LICENSE](./LICENSE.md)).
