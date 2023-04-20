---
description: Enforcing custom API usage restrictions with rate limiting
---

# Custom Rate Limits

Rate limits are an important feature that allows you to control the number of requests made to your API within a specific time window. By implementing rate limits, you can prevent abuse while protecting your resources from being overwhelmed by excessive traffic.

## Getting Started

To set up rate limiting, you only need to provide the `Helicone-RateLimit-Policy` header in your request. This will rate limit all requests made with the specified API key.

The header value should follow this format:

```
[quota];w=[time_window];s=[segment]
```

* `quota` (required): The maximum number of requests allowed within the specified time window.
* `time_window` (required): The length of the time window in seconds. The minimum value is 60.
* `segment` (optional): The rate limiting segment. Can be "user" or a custom property. If left blank, this rate limits all of your requests made with the api key.

{% tabs %}
{% tab title="Curl" %}
<pre class="language-bash"><code class="lang-bash">curl https://oai.hconeai.com/v1/completions \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer YOUR_API_KEY' \
  -H 'Helicone-Property-IP: 111.1.1.1' \
<strong>  -H 'Helicone-RateLimit-Policy: 1000;w=60;s=ip' \
</strong>  -d '{
    "model": "text-davinci-003",
    "prompt": "How do I enable custom rate limit policies?",
}'
</code></pre>
{% endtab %}

{% tab title="Python" %}
<pre class="language-python"><code class="lang-python">openai.api_base = "https://oai.hconeai.com/v1"

openai.Completion.create(
    model="text-davinci-003",
    prompt="How do I enable retries?",
    headers={
<strong>      "Helicone-RateLimit-Policy": "1000;w=60;s=ip_address",
</strong>    }
)
</code></pre>
{% endtab %}

{% tab title="Node" %}
<pre class="language-javascript"><code class="lang-javascript">import { Configuration, OpenAIApi } from "openai";
const configuration = new Configuration({
  apiKey: process.env.OPENAI_API_KEY,
  basePath: "https://oai.hconeai.com/v1",
  baseOptions: {
    headers: {
<strong>      "Helicone-RateLimit-Policy": "1000;w=60;s=ip_address",
</strong>    },
  },
});
const openai = new OpenAIApi(configuration);
</code></pre>
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Fun fact: this policy format is an [IETF standard](https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/) for specifying rate limits! Except for the `segment` field, that's a Helicone special twist :lollipop:
{% endhint %}

### Filtering by segments

You can rate limit for all of your requests made with the API key, by user, or by a custom property. Here's how to set the segment field `s=[segment]`:

* For global rate limiting, leave the `segment` field empty. Your policy can look like `1000;w=60`
* For rate limiting by user, set the segment field to `user`. The user ID must be included as a parameter in the request or in the `helicone-user-id` header, see [user-metrics.md](user-metrics.md "mention") for more details.
* For rate limiting by a custom property, set the segment field to the desired property name in the policy `1000;w=60;s=[property_name]`, and include a corresponding header in the request, formatted as `helicone-property-{property_name}`.

{% hint style="info" %}
The minimum value for the time window is 60. The only unit for the time window field is seconds, so for example, use 60 \* 60 \* 24 = 86400 for a single day.
{% endhint %}

### Examples

The following are a list of example policies to use for `Helicone-RateLimit-Policy`

{% tabs %}
{% tab title="Global Rate Limiting" %}
* Quota: 10k requests
* Time window: 1 hour (3600 seconds)
* Segment: global (default)

Header policy value: `10000;w=3600`
{% endtab %}

{% tab title="Per User Rate Limiting" %}
* Quota: 500k requests
* Time window: 1 day (86400 seconds)
* Segment: user

Header policy value: `500000;w=86400;s=user`

Don't forget to add [user-metrics.md](user-metrics.md "mention")
{% endtab %}

{% tab title="Custom Property Rate Limiting" %}
* Quota: 300 requests
* Time window: 30 minutes (1800 seconds)
* Segment: organization (custom property)

Header policy value: `300;w=1800;s=organization`

Don't forget to set the custom property for organization in the request, see [custom-properties.md](custom-properties.md "mention")
{% endtab %}
{% endtabs %}

### Latency Considerations

Using rate limits adds a small amount of latency to your requests. This feature is deployed with Cloudflare's key-value store, which is a low-latency service that stores data in a small number of centralized data centers and caches that data in Cloudflare's data centers after access. The latency add-on is minimal compared to multi-second OpenAI requests.

### Rate Limit Error

If a request is rate-limited, a 429 rate limit error will be returned.

### Upcoming Features

Very soon, we will support rate limiting by tokens and by cost. Additionally, you will be able to see how close your requests, users, and properties are to hitting their rate limits in the web UI.

### Returned Headers

If rate limiting is active, the following headers will be returned:

* `Helicone-RateLimit-Limit`: The quota for the number of requests allowed in the time window.
* `Helicone-RateLimit-Remaining`: The remaining quota in the current window.
* `Helicone-RateLimit-Reset`: The time remaining in the current window until a request can be made again, specified in seconds. This is only returned if the request was rate limited.
* `Helicone-RateLimit-Policy`: The active rate limit policy.

These headers are only returned if a rate limit policy is active.

{% hint style="info" %}
Test out your rate limit policy in a local environment before deploying to production. Contact `help@helicone.ai` if you have any questions.
{% endhint %}