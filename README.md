# gotify redpanda connect listener

on JSON msg to configured topic produce gotify alert

SAMPLE env vars

    GOTIFY_TOKEN=<xxxxxxxxxxxxxxx>
    GOTIFY_HOST='gotify.xxxxxxx.ts.net'
    GOTIFY_URL="https://gotify.xxxxxxx.ts.net/message"


FINALLY curl the json contained in [sample_msg.json](sample_msg.json)

    curl -s -S \
     --header 'Content-Type: application/json' \
     --header "X-Gotify-Key: ${GOTIFY_TOKEN}"  \
     --data "${GOTIFY_JSON}" \
     "${GOTIFY_URL}"

## how to test your pipeline logic in connect.yaml 

Use the playground app

https://docs.redpanda.com/redpanda-connect/guides/bloblang/playground/

### sample data to get your started

Input
```
[ "one" ]
```

Mapping
```
if this.array().length() > 0 { root.first = this.array().index(0) }
if this.array().length() > 1 { root.second = this.index(1) } else { root.second = "default second value" }
```

Output
```
{
  "first": "one"
}
```

### if .array() checks

The array function turns the data (could be an object or string, number) into an array.  
IF the value is an array returns the array as is
```
this.array()
```

This allows input like:
```
"one"
```

```
[ "one" ]
```

```
[ "one", "two" ]
```
