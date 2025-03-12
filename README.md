# Personal Learning Record Store BB

The Personal Learning Record Store (PLRS) service is a cloud-based platform that empowers individuals to store and manage their learning records centrally. Unlike traditional Learning Record Stores (LRS) owned by training organizations, a PLRS is directly owned by the learner, providing unparalleled control over their personal learning data. With PLRS, users can easily access, download, and reuse their learning records, enhancing data portability and GDPR compliance. This service not only tracks learning activities, achievements, and progress but also facilitates secure and private sharing of this information when desired. Embrace the future of learning with a PLRS—your personal cloud for lifelong education.

## Design Document
See the design document [here](docs/design-document.md).


## Differences from Ralph's original repository

- Better compliance with xapi lrs conformance requirements
- Compatibility with CozyStack backend
- Use of a dependency management system (uv)
- Implementation of a CI pipeline with Github Actions
- [Original Ralph repository](https://github.com/openfun/ralph)


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
    -H "X-Experience-API-Version: 1.0.3" \
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
    -H "X-Experience-API-Version: 1.0.3" \
    http://localhost:8100/xAPI/statements/ \ |
jq
```

> Note that using `jq` is optional in this case, it is used to improve response
> readability. It is not required to install it to run this snippet.


## Example usage

### POST single statement

```bash
cat data/post-single-statement.json | \
curl -Lk \
    --user ralph:secret \
    -X POST \
    -H "Content-Type: application/json" \
    -H "X-Experience-API-Version: 1.0.3" \
    -d @- \
    http://localhost:8100/xAPI/statements/
```

#### Expected output

HTTP response status code: `200 OK`

```json
["72c34cdd-5f03-4040-ad0c-afd8092027a6"]
```

### POST multiple statements

```bash
cat data/post-multiple-statements.json | \
curl -Lk \
    --user ralph:secret \
    -X POST \
    -H "Content-Type: application/json" \
    -H "X-Experience-API-Version: 1.0.3" \
    -d @- \
    http://localhost:8100/xAPI/statements/
```

#### Expected output

HTTP response status code: `200 OK`

```json
["b6484bca-c648-48a3-9cd9-a0ea4887f86f","9f43d4d9-e7f8-42f5-bdea-400dba6e1f14"]
```

### PUT single statement

```bash
cat data/put-single-statement.json | \
curl -Lk \
    --user ralph:secret \
    -X PUT \
    -H "Content-Type: application/json" \
    -H "X-Experience-API-Version: 1.0.3" \
    -d @- \
    "http://localhost:8100/xAPI/statements/?statementId=2654a37f-32c5-4ebd-a650-655b7df326ec"
```

#### Expected output

HTTP response status code: `204 No Content`

### GET all statements

```bash
curl -s \
    --user ralph:secret \
    -X GET \
    -H "Content-Type: application/json" \
    -H "X-Experience-API-Version: 1.0.3" \
    http://localhost:8100/xAPI/statements/ \ |
jq
```

#### Expected output

```json
{
  "statements":[
    {
      "actor":{
        "mbox":"mailto:xapi@adlnet.gov"
      },
      "id":"9f43d4d9-e7f8-42f5-bdea-400dba6e1f14",
      "object":{
        "id":"http://example.adlnet.gov/xapi/example/meeting"
      },
      "verb":{
        "id":"http://adlnet.gov/expapi/verbs/attended",
        "display":{
          "en-US":"attended"
        }
      },
      "timestamp":"2025-01-01T12:00:00.000000",
      "stored":"2025-02-27T08:54:04.589780+00:00",
      "authority":{
        "mbox":"mailto:ralph@example.com",
        "objectType":"Agent"
      }
    },
    {
      "actor":{
        "mbox":"mailto:xapi@adlnet.gov"
      },
      "id":"72c34cdd-5f03-4040-ad0c-afd8092027a6",
      "object":{
        "id":"http://example.adlnet.gov/xapi/example/activity"
      },
      "verb":{
        "id":"http://adlnet.gov/expapi/verbs/created",
        "display":{
          "en-US":"created"
        }
      },
      "timestamp":"2024-12-31T12:00:00.000000",
      "stored":"2025-02-27T08:53:56.283889+00:00",
      "authority":{
        "mbox":"mailto:ralph@example.com",
        "objectType":"Agent"
      }
    },
    {
      "actor":{
        "mbox":"mailto:xapi@adlnet.gov"
      },
      "id":"b6484bca-c648-48a3-9cd9-a0ea4887f86f",
      "object":{
        "id":"http://example.adlnet.gov/xapi/example/activity"
      },
      "verb":{
        "id":"http://adlnet.gov/expapi/verbs/created",
        "display":{
          "en-US":"created"
        }
      },
      "timestamp":"2024-12-31T12:00:00.000000",
      "stored":"2025-02-27T08:54:04.589745+00:00",
      "authority":{
        "mbox":"mailto:ralph@example.com",
        "objectType":"Agent"
      }
    },
    {
      "actor":{
        "mbox":"mailto:xapi@adlnet.gov"
      },
      "id":"2654a37f-32c5-4ebd-a650-655b7df326ec",
      "object":{
        "id":"http://example.adlnet.gov/xapi/example/game"
      },
      "verb":{
        "id":"http://adlnet.gov/expapi/verbs/played",
        "display":{
          "en-US":"played"
        }
      },
      "timestamp":"2024-12-31T12:00:00.000000",
      "stored":"2025-02-27T08:56:18.377837+00:00",
      "authority":{
        "mbox":"mailto:ralph@example.com",
        "objectType":"Agent"
      }
    }
  ]
}
```


### GET statements by id

```bash
curl -s \
    --user ralph:secret \
    -X GET \
    -H "Content-Type: application/json" \
    -H "X-Experience-API-Version: 1.0.3" \
    "http://localhost:8100/xAPI/statements/?statementId=9f43d4d9-e7f8-42f5-bdea-400dba6e1f14" \ |
jq
```

#### Expected output

```json
{
  "actor": {
    "mbox": "mailto:xapi@adlnet.gov"
  },
  "id": "9f43d4d9-e7f8-42f5-bdea-400dba6e1f14",
  "object": {
    "id": "http://example.adlnet.gov/xapi/example/meeting"
  },
  "verb": {
    "id": "http://adlnet.gov/expapi/verbs/attended",
    "display": {
      "en-US": "attended"
    }
  },
  "timestamp": "2025-01-01T12:00:00.000000",
  "stored": "2025-02-27T08:54:04.589780+00:00",
  "authority": {
    "mbox": "mailto:ralph@example.com",
    "objectType": "Agent"
  }
}
```

### GET statements by verb

```bash
curl -s \
    --user ralph:secret \
    -X GET \
    -H "Content-Type: application/json" \
    -H "X-Experience-API-Version: 1.0.3" \
    "http://localhost:8100/xAPI/statements/?verb=http%3A%2F%2Fadlnet.gov%2Fexpapi%2Fverbs%2Fcreated" \ |
jq
```

#### Expected output

```json
{
  "statements": [
    {
      "actor": {
        "mbox": "mailto:xapi@adlnet.gov"
      },
      "id": "72c34cdd-5f03-4040-ad0c-afd8092027a6",
      "object": {
        "id": "http://example.adlnet.gov/xapi/example/activity"
      },
      "verb": {
        "id": "http://adlnet.gov/expapi/verbs/created",
        "display": {
          "en-US": "created"
        }
      },
      "timestamp": "2024-12-31T12:00:00.000000",
      "stored": "2025-02-27T08:53:56.283889+00:00",
      "authority": {
        "mbox": "mailto:ralph@example.com",
        "objectType": "Agent"
      }
    },
    {
      "actor": {
        "mbox": "mailto:xapi@adlnet.gov"
      },
      "id": "b6484bca-c648-48a3-9cd9-a0ea4887f86f",
      "object": {
        "id": "http://example.adlnet.gov/xapi/example/activity"
      },
      "verb": {
        "id": "http://adlnet.gov/expapi/verbs/created",
        "display": {
          "en-US": "created"
        }
      },
      "timestamp": "2024-12-31T12:00:00.000000",
      "stored": "2025-02-27T08:54:04.589745+00:00",
      "authority": {
        "mbox": "mailto:ralph@example.com",
        "objectType": "Agent"
      }
    }
  ]
}
```

### GET statements by activity

```bash
curl -s \
    --user ralph:secret \
    -X GET \
    -H "Content-Type: application/json" \
    -H "X-Experience-API-Version: 1.0.3" \
    "http://localhost:8100/xAPI/statements/?activity=http%3A%2F%2Fexample.adlnet.gov%2Fxapi%2Fexample%2Fmeeting" \ |
jq
```

#### Expected output

```json
{
  "statements": [
    {
      "actor": {
        "mbox": "mailto:xapi@adlnet.gov"
      },
      "id": "9f43d4d9-e7f8-42f5-bdea-400dba6e1f14",
      "object": {
        "id": "http://example.adlnet.gov/xapi/example/meeting"
      },
      "verb": {
        "id": "http://adlnet.gov/expapi/verbs/attended",
        "display": {
          "en-US": "attended"
        }
      },
      "timestamp": "2025-01-01T12:00:00.000000",
      "stored": "2025-02-27T08:54:04.589780+00:00",
      "authority": {
        "mbox": "mailto:ralph@example.com",
        "objectType": "Agent"
      }
    }
  ]
}
```

### GET statements since

```bash
curl -s \
    --user ralph:secret \
    -X GET \
    -H "Content-Type: application/json" \
    -H "X-Experience-API-Version: 1.0.3" \
    "http://localhost:8100/xAPI/statements/?since=2025-01-01T09:00:00.000000" \ |
jq
```

#### Expected output

```json
{
  "statements": [
    {
      "actor": {
        "mbox": "mailto:xapi@adlnet.gov"
      },
      "id": "9f43d4d9-e7f8-42f5-bdea-400dba6e1f14",
      "object": {
        "id": "http://example.adlnet.gov/xapi/example/meeting"
      },
      "verb": {
        "id": "http://adlnet.gov/expapi/verbs/attended",
        "display": {
          "en-US": "attended"
        }
      },
      "timestamp": "2025-01-01T12:00:00.000000",
      "stored": "2025-02-27T08:54:04.589780+00:00",
      "authority": {
        "mbox": "mailto:ralph@example.com",
        "objectType": "Agent"
      }
    }
  ]
}
```

### GET statements until

```bash
curl -s \
    --user ralph:secret \
    -X GET \
    -H "Content-Type: application/json" \
    -H "X-Experience-API-Version: 1.0.3" \
    "http://localhost:8100/xAPI/statements/?until=2024-12-31T23:00:00.000000" \ |
jq
```

#### Expected output

```json
{
  "statements":[
    {
      "actor":{
        "mbox":"mailto:xapi@adlnet.gov"
      },
      "id":"72c34cdd-5f03-4040-ad0c-afd8092027a6",
      "object":{
        "id":"http://example.adlnet.gov/xapi/example/activity"
      },
      "verb":{
        "id":"http://adlnet.gov/expapi/verbs/created",
        "display":{
          "en-US":"created"
        }
      },
      "timestamp":"2024-12-31T12:00:00.000000",
      "stored":"2025-02-27T08:53:56.283889+00:00",
      "authority":{
        "mbox":"mailto:ralph@example.com",
        "objectType":"Agent"
      }
    },
    {
      "actor":{
        "mbox":"mailto:xapi@adlnet.gov"
      },
      "id":"b6484bca-c648-48a3-9cd9-a0ea4887f86f",
      "object":{
        "id":"http://example.adlnet.gov/xapi/example/activity"
      },
      "verb":{
        "id":"http://adlnet.gov/expapi/verbs/created",
        "display":{
          "en-US":"created"
        }
      },
      "timestamp":"2024-12-31T12:00:00.000000",
      "stored":"2025-02-27T08:54:04.589745+00:00",
      "authority":{
        "mbox":"mailto:ralph@example.com",
        "objectType":"Agent"
      }
    },
    {
      "actor":{
        "mbox":"mailto:xapi@adlnet.gov"
      },
      "id":"2654a37f-32c5-4ebd-a650-655b7df326ec",
      "object":{
        "id":"http://example.adlnet.gov/xapi/example/game"
      },
      "verb":{
        "id":"http://adlnet.gov/expapi/verbs/played",
        "display":{
          "en-US":"played"
        }
      },
      "timestamp":"2024-12-31T12:00:00.000000",
      "stored":"2025-02-27T08:56:18.377837+00:00",
      "authority":{
        "mbox":"mailto:ralph@example.com",
        "objectType":"Agent"
      }
    }
  ]
}
```

## Unit testing

### Setup test environment

Build the project : 

```bash
docker compose build 
```

Bootstrap the project : 

```bash
make bootstrap
```

Run ralph : 

```bash
make run
```

### Run tests

```bash
bin/pytest --ignore=tests/api 
```

### Expected results

```
1523 passed, 6 skipped, 291 warnings
```

## Component-level testing


### Setup test environment

Build the project : 

```bash
docker compose build 
```

Bootstrap the project : 

```bash
make bootstrap
```

Run ralph : 

```bash
make run
```

### Run tests

```bash
bin/pytest tests/api 
```

### Expected results

```
422 passed, 342 warnings 
```

## License

This work is released under the MIT License (see [LICENSE](./LICENSE.md)).
