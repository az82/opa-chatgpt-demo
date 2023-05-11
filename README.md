# Delegating OPA decisions to ChatGPT

This is a silly little example how you could delegate policy decisions to ChatGPT.
While this is intended to be taken lightly, there are valid use cases.
For example, you could use ChatGPT for assessing the risk of critical requests.

## Prerequisites

* [Open Policy Agent](https://github.com/open-policy-agent/opa) CLI installed
* [ChatGPT API access](https://platform.openai.com/)
* A `data.json` file providing your ChatGPT API key:

  ```json
  { "openai_api_key": "YOUR-OPENAPI-KEY" }
  ```

## 1. Run OPA

```sh
opa run data.json
```

## 2. Implement The Policy

```rego
response := http.send({
    "url": "https://api.openai.com/v1/chat/completions",
    "method": "POST",
    "headers": {
        "Content-Type": "application/json",
        "Authorization": concat(" ", ["Bearer", data.openai_api_key])
    },
    "body": {
     "model": "gpt-3.5-turbo",
     "messages": [
        { "role": "system",  "content": "You are an bouncer. If a user has a role named 'developer', respond only with 'ACCESS GRANTED'. Otherwise, respond by merrily insulting the user." },
        { "role": "user", "content": concat(" ", ["May I enter? I have the roles", concat(", ", input.roles)])}
    ]
   }
})
default allowed := false
allowed {
    contains(response.body.choices[0].message.content, "ACCESS GRANTED")
}
```

## 3. Add the input document

```rego
input := { "roles": ["user"] }
```

## 4. Inspect the result

```rego
allowed
response.body.choices[0].message.content
```

## 5. Test with other data

```rego
input := { "roles": ["developer"] }
allowed
```
