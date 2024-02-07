---
title: ChatGPT function call 사용해보기
author: sunshine@ptokos.com
categories: [ chatgpt, functioncall ]
---


## Function calling
OpenAI [Function calling](https://platform.openai.com/docs/guides/function-calling) 문서를 보자.

> In an API call, you can describe functions and have the model intelligently choose to output a JSON object containing arguments to call one or many functions.
> The Chat Completions API does not call the function; instead, the model generates JSON that you can use to call the function in your code.

즉 API 호출에서 함수를 설명하고 모델이 하나 이상의 함수를 호출할 수 있는 인자를 포함하는 JSON 객체를 출력하도록 할 수 있다.


## 기존

기존에는 아래와 같이 JSON 형태로 알려달라고 또 원하는 포멧은 어떤지 예시를 통해 설명했다.
user 질문 앞뒤로 assistant 가 있는데 이유는 이렇게 해야 순수 JSON 형태로 출력할 확률이 높다는 경험상 이유가 있다.

```typescript
const systemPrompt =
    `Create a summary in 5 lines and the 10 categories. Each field must have a value.
    Do not include any explanations, only provide a  RFC8259 compliant JSON response  following this format without deviation.
`;

const requestCompletion = async (prompt: string) => {
    return openai.chat.completions.create({
        model: "gpt-3.5-turbo-16k",
        messages: [
            {
                "role": "system",
                "content": systemPrompt,
            },
            {
                "role": "assistant",
                "content": `The summary does not exceed 7 lines.    
                {
                    "ko": {
                        "categories": [],
                        "summary": ""
                    },
                    "en": {
                        "categories": [],
                        "summary": ""
                    }
                }`
            },
            {
                "role": "user",
                "content": prompt
            },
            {
                "role": "assistant",
                "content": `
                The summary does not exceed 7 lines.     
                 {
                    "ko": {
                        "categories": [],
                        "summary": ""
                    },
                    "en": {
                        "categories": [],
                        "summary": ""
                    }
                }`
            },
        ],
        temperature: 0,
        max_tokens: 2000,
        top_p: 1,
        frequency_penalty: 0,
        presence_penalty: 0,
    });
}
```


## Function calling 사용
확실히 JSON 형태로 출력하는 것이 더 명확하다. 추후 여러 정보들이 더 필요할 때 확장에 편리하겠다.

```typescript
const systemPrompt =
    `Summarize the collected articles and categories. Create a summary in 5 lines and the 10 categories.
`;

const requestCompletion = async (prompt: string) => {
    return openai.chat.completions.create({
        model: "gpt-3.5-turbo-16k",
        messages: [
            {
                "role": "system",
                "content": systemPrompt,
            },
            {
                "role": "assistant",
                "content": `The summary does not exceed 7 lines.`
            },
            {
                "role": "user",
                "content": prompt
            },
        ],
        functions: [
            {
                name: "summary_and_categories",
                parameters: {
                    type: "object",
                    properties: {
                        ko: {
                            type: "object",
                            properties: {
                                categories: {
                                    type: "array",
                                    items: {
                                        type: "string"
                                    }
                                },
                                summary: {
                                    type: "string"
                                }
                            },
                            required: ["categories", "summary"]
                        },
                        en: {
                            type: "object",
                            properties: {
                                categories: {
                                    type: "array",
                                    items: {
                                        type: "string"
                                    }
                                },
                                summary: {
                                    type: "string"
                                }
                            },
                            required: ["categories", "summary"]
                        }
                    },
                    required: ["ko", "en"]
                }
            }
        ],
        function_call: {name: "summary_and_categories"},
        temperature: 0,
        max_tokens: 2000,
        top_p: 1,
        frequency_penalty: 0,
        presence_penalty: 0,
    });
}
```









