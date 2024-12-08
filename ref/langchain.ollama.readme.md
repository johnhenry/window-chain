https://developer.chrome.com/docs/extensions/ai/prompt-api#model_capabilities

# Langchain Ollama

This notebook shows how to use Langchain with Ollama.

## Instantiation

First, instantiate a `ChatOllama` object.

```javascript
import { ChatOllama } from "@langchain/ollama";

const llm = new ChatOllama({
  model: "llama2",
  temperature: 0,
  maxRetries: 2,
  // other params...
});
```

## Invocation

Invoke the model using the `invoke` method.

```javascript
const aiMsg = await llm.invoke([
  ["system", "You are a helpful assistant that translates English to French. Translate the user sentence."],
  ["human", "I love programming."],
]);

console.log(aiMsg.content);
```

## Chaining

Chain the model with a prompt template.

```javascript
import { ChatPromptTemplate } from "@langchain/core/prompts";

const prompt = ChatPromptTemplate.fromMessages([
  ["system", "You are a helpful assistant that translates {input_language} to {output_language}."],
  ["human", "{input}"],
]);

const chain = prompt.pipe(llm);
const result = await chain.invoke({
  input_language: "English",
  output_language: "German",
  input: "I love programming.",
});

console.log(result.content);
```

## Tools

Ollama supports native tool calling for a subset of models.

```javascript
import { tool } from "@langchain/core/tools";
import { ChatOllama } from "@langchain/ollama";
import { z } from "zod";

const weatherTool = tool((_) => "The weather is weatherin", {
  name: "get_current_weather",
  description: "Get the current weather in a given location",
  schema: z.object({
    location: z.string().describe("The city and state, e.g. San Francisco, CA"),
  }),
});

const llmForTool = new ChatOllama({ model: "llama2" }); // Replace with a tool-compatible model
const llmWithTools = llmForTool.bindTools([weatherTool]);

const resultFromTool = await llmWithTools.invoke(
  "What's the weather like today in San Francisco? Ensure you use the 'get_current_weather' tool."
);

console.log(resultFromTool);
```

### `.withStructuredOutput`

For models that support tool calling, use `.withStructuredOutput()` to get structured output.

```javascript
import { ChatOllama } from "@langchain/ollama";
import { z } from "zod";

const llmForWSO = new ChatOllama({ model: "llama2" }); // Replace with a tool-compatible model
const schemaForWSO = z.object({
  location: z.string().describe("The city and state, e.g. San Francisco, CA"),
});

const llmWithStructuredOutput = llmForWSO.withStructuredOutput(schemaForWSO, {
  name: "get_current_weather",
});

const resultFromWSO = await llmWithStructuredOutput.invoke(
  "What's the weather like today in San Francisco? Ensure you use the 'get_current_weather' tool."
);

console.log(resultFromWSO);
```

## JSON Mode

Ollama supports a JSON mode for all chat models.

```javascript
import { ChatOllama } from "@langchain/ollama";
import { ChatPromptTemplate } from "@langchain/core/prompts";

const promptForJsonMode = ChatPromptTemplate.fromMessages([
  ["system", `You are an expert translator. Format all responses as JSON objects with two keys: "original" and "translated".`],
  ["human", `Translate "{input}" into {language}.`],
]);

const llmJsonMode = new ChatOllama({
  model: "llama2",
  format: "json",
});

const chainForJsonMode = promptForJsonMode.pipe(llmJsonMode);

const resultFromJsonMode = await chainForJsonMode.invoke({
  input: "I love programming",
  language: "German",
});

console.log(resultFromJsonMode);
```

## Multimodal Models

Ollama supports open source multimodal models like LLaVA.

```javascript
import { ChatOllama } from "@langchain/ollama";
import { HumanMessage } from "@langchain/core/messages";
import * as fs from "node:fs/promises";

const imageData = await fs.readFile("./path/to/your/image.jpg"); // Replace with your image path
const llmForMultiModal = new ChatOllama({ model: "llava" });
const multiModalRes = await llmForMultiModal.invoke([
  new HumanMessage({
    content: [
      { type: "text", text: "What is in this image?" },
      { type: "image_url", image_url: `data:image/jpeg;base64,${imageData.toString("base64")}` },
    ],
  }),
]);

console.log(multiModalRes);
