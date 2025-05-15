# gotify redpanda connect listener

Redpand-connect offers an alternative framework to kafka connect and is based on redpanda tech.

I've found it to be easier, faster to dev/test with better tooling.

## this app

    kafka connect application 

    Consumes JSON strings (OR arrays) produced to the configured kafka topic in /connect.yaml

    The strings are transformed into gotify messages and  POST'd to the gotify server url in docker-compose.yml

    The gotify server on POST, sends an alert to members of application configured for the token, and message finally makes it to my phone.

[docker-compose.yml](docker-compose.yml) 

### SAMPLE .env docker-compose file var that need to be set

    GOTIFY_TOKEN=<xxxxxxxxxxxxxxx>
    GOTIFY_URL="https://gotify.xxxxxxx.ts.net/message"

The token is associated with a gotify 'app' which allows you to segment your messages a bit.

### SAMPLE gotify POST using curl

[sample_gotify_msg.json](sample_gotify_msg.json)

    curl -s -S \
     --header 'Content-Type: application/json' \
     --header "X-Gotify-Key: ${GOTIFY_TOKEN}"  \
     --data "${GOTIFY_JSON}" \
     "${GOTIFY_URL}"

## modifying redpanda connect pipeline logic

To dev/test the pipeline logic in [connect.yaml](connect.yaml) and the *-resource.yaml files

I suggest using the redpanda playground app

https://docs.redpanda.com/redpanda-connect/guides/bloblang/playground/

Just paste sample message data, add the logic to be tested, which eventually would find it's way into a **<name>-resources.yaml** file. 

And test away.


I suggest isolating your pipeline logic into multiple **-resources.yaml** files and mount them at the root of docker container.

This app has three files

* /[connect.yaml](connect.yaml)
* /[mapping-resources.yaml](mapping-resources.yaml)
* /[gotify-resources.yaml](gotify-resources.yaml)

Finally, any **-resources.yaml** files added need to be configured in docker-compose.yml command:

###  pipline data/mappings and how it works at a high level

Check out the docs for the redpanda connect mapping language, called bloblang (yes, naming things is hard)

https://docs.redpanda.com/redpanda-connect/guides/bloblang/about/

And play around with some sample data and scripts using the playground app linked above

Sample Input
```
[ "one" ]
```

Sample Mapping
```
if this.array().length() > 0 { root.first = this.array().index(0) }
if this.array().length() > 1 { root.second = this.index(1) } else { root.second = "default second value" }
```

Expected Output
```
{
  "first": "one"
}
```

#### things to note in the bloblang above
##### this. and root.

In this, and well any context if I'm being honest -

**this.** refers to **WHATEVER** the caller is.  

When a topline pipeline step is invoked, **this.** would refer to the message and meta in the pipeline.

**root.** will generally refer to the transformed resulting JSON obect

##### this.array() checks

The array() function when applied to a message in the pipeline

    this.array()

allows you to now consinder the the data passed in the body (could be an object or string, number) as an array.

WHEN the value is **already** an array, a reference to the original array is returned

This allows input like:
```
"one"
```

becomes 
```
[ "one" ]
```

AND an array such as -
```
[ "one", "two" ]
```

Would be returned as the original array. 

Perty handy for considering / producing single JS strings to the topic, then using length checks in the bloblang.
