
# **A Comprehensive Guide to LLM Inference with the GitHub Models REST API and the Deno Runtime**


## **Section 1: An Introduction to the GitHub Models API**

The GitHub Models REST API represents a significant expansion of GitHub's platform, moving beyond code hosting and DevOps into the realm of AI development infrastructure. It provides developers with programmatic access to a curated selection of powerful Large Language Models (LLMs). This guide offers a comprehensive, expert-level documentation for leveraging this API, with a specific focus on implementation within the modern, security-conscious Deno runtime.


### **1.1 Architectural Overview: The Dual Roles of Catalog and Inference**

The architecture of the GitHub Models API is fundamentally bifurcated into two distinct, yet complementary, functional areas: discovery and execution.<sup>1</sup> This design is exposed through two primary sets of endpoints: the Models Catalog and Models Inference.<sup>1</sup>



1. **Models Catalog**: This component serves as a queryable registry of all available models on the platform. Its purpose is to allow applications to programmatically discover which models are available, who their publishers are, and what their specific capabilities are, such as supported modalities and rate limits.<sup>1</sup>
2. **Models Inference**: This is the execution engine of the API. Once a suitable model has been identified via the catalog, the inference endpoints are used to submit prompts and receive generated completions.<sup>1</sup>

This separation of concerns is a deliberate architectural choice that promotes a robust and decoupled workflow. An application can first query the catalog to dynamically populate a list of models for a user, filter them based on required features (e.g., image input), and then use the selected model's unique identifier to perform an inference. This design is characteristic of enterprise-grade Platform-as-a-Service (PaaS) offerings, signaling GitHub's intention for this API to be a foundational, scalable service for serious application development.<sup>4</sup>


### **1.2 Core Capabilities: Beyond Basic Chat**

The GitHub Models API is not a single-model endpoint but rather a powerful aggregator, providing a unified interface to a diverse range of state-of-the-art models from leading AI providers, including OpenAI, DeepSeek, Microsoft, and Llama.<sup>3</sup> This abstraction layer significantly simplifies development by eliminating the need to integrate with multiple, disparate third-party APIs, each with its own authentication scheme and SDK.

The API's capabilities are extensive and designed for building sophisticated AI applications <sup>3</sup>:



* **Streaming and Non-Streaming Completions**: The API natively supports both synchronous (non-streaming) and asynchronous (streaming) responses.<sup>3</sup> Streaming is essential for creating interactive, real-time experiences like chatbots, where perceived latency is critical. Non-streaming is suitable for backend tasks, batch processing, or any scenario where a complete response is required before proceeding.
* **Advanced Parameter Control**: Developers have granular control over the inference process through a rich set of parameters. This includes foundational controls like temperature for creativity and max_tokens for response length, as well as advanced options like frequency_penalty and presence_penalty to reduce repetition.<sup>3</sup>
* **Tool Integration**: A first-class tools parameter allows developers to define functions that the model can request to call. This enables the creation of powerful AI agents that can interact with external systems to retrieve information or perform actions.<sup>3</sup>
* **Organizational Attribution**: The API includes specific endpoints for attributing usage and tracking to a GitHub Organization. This is a critical feature for enterprises needing to monitor consumption and manage costs within their existing GitHub ecosystem.<sup>3</sup>

The inclusion of these advanced features from the outset indicates that the API is engineered not just for simple prompt-response interactions, but for the development of complex, stateful, and extensible AI-powered applications. The API's design as a model aggregator, combined with its deep integration into GitHub's organizational and billing structures (rate limits are tied to GitHub Copilot subscription tiers), points to a clear strategy.<sup>8</sup> GitHub is positioning itself as a central, managed hub for enterprise AI development, leveraging its vast existing footprint in the software development lifecycle. Developers utilizing this API are plugging into this broader ecosystem, a factor that may influence future costs, governance, and feature availability.<sup>3</sup>


### **1.3 Understanding the API Surface: Key Endpoints and Versioning**

To interact with the API, developers must be familiar with its primary endpoints and adhere to its versioning scheme. The core endpoints are <sup>1</sup>:



* GET /catalog/models: Lists all available models in the catalog.
* POST /inference/chat/completions: Runs a standard inference request.
* POST /orgs/{org}/inference/chat/completions: Runs an inference request attributed to a specific organization.

The GitHub REST API is explicitly versioned to ensure stability and backward compatibility. The current version that must be used for all requests to the Models API is 2022-11-28.<sup>2</sup> This version must be specified in every API call via the

X-GitHub-Api-Version header. Failure to do so may result in unexpected behavior or errors, as the request would fall back to a default version that may not support the Models API features.<sup>11</sup>


## **Section 2: Authentication and Secure Environment Setup**

Securely authenticating with the GitHub Models API and configuring a safe development environment are non-negotiable prerequisites for any application. This section provides a security-first guide to preparing a Deno environment for API interaction, emphasizing the principle of least privilege.


### **2.1 The models:read Permission: A Foundational Requirement**

All interactions with the GitHub Models API, whether listing models from the catalog or running inference, are governed by a single, fine-grained permission: models:read.<sup>3</sup> This permission can be granted to a fine-grained Personal Access Token (PAT) or a token generated by a GitHub App.<sup>12</sup> For individual developers and scripting, a PAT is the most direct method. The specificity of this permission is a key security feature; a compromised token with only

models:read access cannot be used to modify repositories, manage organization settings, or perform other sensitive actions.


### **2.2 Generating and Using Fine-Grained Personal Access Tokens (PATs)**

A fine-grained PAT with the necessary permission must be created through the GitHub user interface.

**Steps to Create a PAT:**



1. Navigate to your GitHub **Settings**.
2. In the left sidebar, click **Developer settings**.
3. Click **Personal access tokens**, then select **Fine-grained tokens**.
4. Click **Generate new token**.
5. Provide a descriptive **Token name** (e.g., "Deno Models API Client").
6. Set an **Expiration** for the token. Shorter-lived tokens are more secure.
7. Under **Repository access**, you can typically leave "All repositories" or select specific ones if your use case is tied to repository context, although the Models API itself is not a repository-specific resource.
8. Under **Permissions**, locate the **GitHub Models** permission set and select models:read from the dropdown. This is a top-level permission, distinct from repository permissions.<sup>4</sup>
9. Click **Generate token**.

**CRITICAL WARNING:** Upon generation, the token will be displayed once. It must be copied and stored securely immediately. This token is equivalent to a password and grants access to your account's Models API entitlements. It should **never** be hardcoded into source code, committed to version control, or exposed in client-side applications.<sup>12</sup>


### **2.3 Configuring the Deno Environment**

Deno's security-by-default model requires explicit permission for potentially sensitive operations. This, combined with best practices for credential management, creates a robust development environment.


#### **2.3.1 Managing Permissions**

When running a Deno script that interacts with the API, specific permission flags must be provided on the command line. This adheres to the principle of least privilege, ensuring the script can only perform the actions it absolutely needs.



* --allow-net=api.github.com: This flag grants the script permission to make outbound network requests, but *only* to the api.github.com domain. Any attempt to contact another domain will be blocked.
* --allow-env=GITHUB_TOKEN: This flag allows the script to read the value of a *specific* environment variable, in this case, GITHUB_TOKEN. The script will be denied access to any other environment variables.<sup>14</sup>


#### **2.3.2 Securely Handling API Tokens with .env Files**

The standard and secure method for managing secrets like API tokens in a Deno project is to use environment variables, often loaded from a .env file.



1. **Create a .env file** in the root of your project. This file should be added to your .gitignore to prevent it from being committed to version control.
2. **Add the token** to the .env file: \
Code snippet \
GITHUB_TOKEN="ghp_YourSecretTokenGoesHere" \

3. **Load the environment variable**. Deno has built-in support for loading .env files using the --env-file flag when running a script <sup>15</sup>: \
Bash \
deno run --allow-net=api.github.com --allow-env=GITHUB_TOKEN --env-file main.ts \

4. **Access the token in your code**. The script can now securely access the token using the Deno.env API.<sup>14</sup> \
TypeScript \
// main.ts \
const apiToken = Deno.env.get("GITHUB_TOKEN"); \
 \
if (!apiToken) { \
  console.error("Error: GITHUB_TOKEN environment variable not set."); \
  Deno.exit(1); \
} \
 \
// Use apiToken for authentication... \


This combination of GitHub's fine-grained tokens and Deno's granular permission system creates a highly secure-by-default development environment. The API token is narrowly scoped to a single capability (models:read), and the Deno process is sandboxed, permitted only to read that one specific token and communicate with the designated API endpoint.<sup>3</sup> This layered security model provides strong protection against common vulnerabilities, such as a compromised dependency attempting to exfiltrate other secrets or connect to malicious domains.


## **Section 3: Discovering Models: The Catalog API**

The first practical step in using the GitHub Models API is to discover what models are available for inference. The Catalog API provides a dedicated endpoint for this purpose, allowing applications to dynamically query and filter the list of supported models.


### **3.1 Constructing a Request to List All Available Models**

To retrieve the model catalog, a GET request must be sent to the appropriate endpoint with the correct headers.



* **Endpoint**: GET https://api.github.com/catalog/models <sup>2</sup>
* **Method**: GET <sup>11</sup>
* **Required Headers**: Every request to the catalog endpoint must include three specific headers to ensure it is properly authenticated, routed, and versioned.
    * Accept: application/vnd.github+json: Specifies that the client expects a JSON-formatted response.<sup>2</sup>
    * Authorization: Bearer &lt;YOUR-TOKEN>: Provides the fine-grained Personal Access Token for authentication. &lt;YOUR-TOKEN> must be replaced with the actual token generated in Section 2.<sup>2</sup>
    * X-GitHub-Api-Version: 2022-11-28: Pins the request to a specific, stable version of the API, preventing future breaking changes from affecting the application.<sup>2</sup>


### **3.2 Interpreting the Model Catalog Response**

A successful request to the catalog endpoint will return an HTTP 200 OK status code and a JSON body containing an array of model objects.<sup>2</sup> Each object in this array represents a single model available for inference and provides detailed metadata. Understanding these fields is critical for selecting the correct model for a given task.


<table>
  <tr>
   <td>Field
   </td>
   <td>Type
   </td>
   <td>Description
   </td>
   <td>Example
   </td>
  </tr>
  <tr>
   <td>id
   </td>
   <td>string
   </td>
   <td>The unique identifier for the model. This value is <strong>required</strong> for the model parameter in an inference request.
   </td>
   <td>openai/gpt-4.1
   </td>
  </tr>
  <tr>
   <td>name
   </td>
   <td>string
   </td>
   <td>The human-readable name of the model.
   </td>
   <td>OpenAI GPT-4.1
   </td>
  </tr>
  <tr>
   <td>publisher
   </td>
   <td>string
   </td>
   <td>The organization or entity that published the model.
   </td>
   <td>OpenAI
   </td>
  </tr>
  <tr>
   <td>summary
   </td>
   <td>string
   </td>
   <td>A brief description of the model's capabilities and intended use.
   </td>
   <td>gpt-4.1 outperforms gpt-4o across the board...
   </td>
  </tr>
  <tr>
   <td>rate_limit_tier
   </td>
   <td>string
   </td>
   <td>The performance and rate limit category for the model (e.g., high, low). This corresponds to usage limits tied to GitHub Copilot subscriptions.
   </td>
   <td>high
   </td>
  </tr>
  <tr>
   <td>supported_input_modalities
   </td>
   <td>array of strings
   </td>
   <td>An array listing the types of input the model can process.
   </td>
   <td>["text", "image", "audio"]
   </td>
  </tr>
  <tr>
   <td>supported_output_modalities
   </td>
   <td>array of strings
   </td>
   <td>An array listing the types of output the model can generate.
   </td>
   <td>["text"]
   </td>
  </tr>
  <tr>
   <td>tags
   </td>
   <td>array of strings
   </td>
   <td>A set of descriptive tags that can be used for filtering and categorization.
   </td>
   <td>["multipurpose", "multilingual"]
   </td>
  </tr>
</table>


Table 3.1: Model Catalog Response Fields. Data sourced from.<sup>2</sup>

The id field is the most important piece of data for subsequent API calls, as it directly maps to the model parameter in the inference endpoint. The supported_input_modalities field is also crucial for building applications that might need to handle more than just text, such as vision-enabled chatbots.


### **3.3 Practical Deno Implementation: A Reusable Function to Fetch and Filter Models**

The following Deno script demonstrates how to create a reusable and strongly-typed function to fetch the model catalog and filter the results.


    TypeScript

// lib/github_models_api.ts \
 \
// Define a TypeScript interface for strong typing of the model object \
export interface GitHubModel { \
  id: string; \
  name: string; \
  publisher: string; \
  summary: string; \
  rate_limit_tier: string; \
  supported_input_modalities: string; \
  supported_output_modalities: string; \
  tags: string; \
} \
 \
const API_BASE_URL = "https://api.github.com"; \
const API_VERSION = "2022-11-28"; \
 \
/** \
 * Fetches the list of available models from the GitHub Models Catalog API. \
 * @param apiToken A fine-grained GitHub PAT with 'models:read' permission. \
 * @returns A promise that resolves to an array of GitHubModel objects. \
 */ \
export async function listModels(apiToken: string): Promise&lt;GitHubModel> { \
  const headers = new Headers({ \
    "Accept": "application/vnd.github+json", \
    "Authorization": `Bearer ${apiToken}`, \
    "X-GitHub-Api-Version": API_VERSION, \
    "User-Agent": "Deno-GitHub-Models-Client/1.0", \
  }); \
 \
  const response = await fetch(`${API_BASE_URL}/catalog/models`, { \
    method: "GET", \
    headers: headers, \
  }); \
 \
  if (!response.ok) { \
    // A more robust error handling mechanism will be detailed in Section 8 \
    throw new Error(`API request failed: ${response.status} ${response.statusText}`); \
  } \
 \
  const models: GitHubModel = await response.json(); \
  return models; \
} \
 \
// Example usage in another file (e.g., main.ts) \
// To run this: deno run --allow-net=api.github.com --allow-env=GITHUB_TOKEN --env-file main.ts \
/* \
import { listModels } from "./lib/github_models_api.ts"; \
 \
const token = Deno.env.get("GITHUB_TOKEN"); \
if (!token) { \
  console.error("GITHUB_TOKEN is not set."); \
  Deno.exit(1); \
} \
 \
try { \
  const allModels = await listModels(token); \
  console.log("All Available Models:", allModels.length); \
 \
  // Example: Filter for models that support image input \
  const visionModels = allModels.filter(model =>  \
    model.supported_input_modalities.includes("image") \
  ); \
 \
  console.log("\nVision-Capable Models:"); \
  visionModels.forEach(model => console.log(`- ${model.id} (${model.name})`)); \
 \
} catch (error) { \
  console.error("Failed to fetch models:", error.message); \
} \
*/ \


This example encapsulates the API call in a clean function, uses a TypeScript interface for type safety and autocompletion, and demonstrates a practical use case of filtering the results to find models with specific capabilities.


## **Section 4: Executing Foundational Inference: Non-Streaming Chat Completions**

Once a model has been selected from the catalog, the next step is to perform inference. This section covers the most fundamental interaction: a synchronous, non-streaming chat completion request. This pattern is ideal for backend processing, summarization, classification, or any task where the entire response is needed before the application proceeds.


### **4.1 The Anatomy of an Inference Request**

A non-streaming inference request is made by sending an HTTP POST request with a JSON payload to the chat/completions endpoint.



* **Endpoint**: POST https://api.github.com/inference/chat/completions <sup>3</sup>
* **Method**: POST <sup>11</sup>
* **Body Parameters**: The JSON body of the request contains the instructions and context for the model.

<table>
  <tr>
   <td>
Parameter
   </td>
   <td>Type
   </td>
   <td>Required?
   </td>
   <td>Description
   </td>
  </tr>
  <tr>
   <td>model
   </td>
   <td>string
   </td>
   <td>Yes
   </td>
   <td>The ID of the model to use, obtained from the Catalog API (e.g., openai/gpt-4.1).
   </td>
  </tr>
  <tr>
   <td>messages
   </td>
   <td>array of objects
   </td>
   <td>Yes
   </td>
   <td>An array of message objects representing the conversation history. Each object must have role and content keys.
   </td>
  </tr>
  <tr>
   <td>max_tokens
   </td>
   <td>integer
   </td>
   <td>No
   </td>
   <td>The maximum number of tokens to generate in the completion.
   </td>
  </tr>
  <tr>
   <td>temperature
   </td>
   <td>number
   </td>
   <td>No
   </td>
   <td>A value between 0 and 2 that controls randomness. Higher values make the output more random, while lower values make it more deterministic.
   </td>
  </tr>
  <tr>
   <td>stream
   </td>
   <td>boolean
   </td>
   <td>No
   </td>
   <td>Must be set to false or omitted for a non-streaming request. Default is false.
   </td>
  </tr>
</table>


Table 4.1: Core Inference Request Body Parameters. Data sourced from.<sup>3</sup>

The messages array is the core of the prompt. It typically follows a structured pattern to guide the model's behavior effectively. A message object has two properties:



* role: A string indicating who is speaking. Common roles are system (provides high-level instructions for the model's persona and behavior), user (represents the end-user's input), and assistant (represents previous responses from the model).<sup>3</sup>
* content: A string containing the text of the message.<sup>3</sup>

A typical conversation starts with a system message, followed by alternating user and assistant messages to provide conversational context.


### **4.2 Building and Sending a POST Request with Deno's fetch**

Deno's global fetch function, which is compliant with the web standard, is used to send the POST request. The key is to correctly construct the RequestInit object.


    TypeScript

// lib/github_models_api.ts (continued) \
 \
// Interface for the chat completion request body \
export interface ChatCompletionRequest { \
  model: string; \
  messages: { role: string; content: string }; \
  max_tokens?: number; \
  temperature?: number; \
  stream?: false; // Explicitly false for this function \
} \
 \
/** \
 * Performs a non-streaming chat completion request. \
 * @param apiToken A fine-grained GitHub PAT with 'models:read' permission. \
 * @param requestBody The body of the chat completion request. \
 * @returns A promise that resolves to the API response object. \
 */ \
export async function createChatCompletion(apiToken: string, requestBody: ChatCompletionRequest) { \
  const headers = new Headers({ \
    "Accept": "application/vnd.github+json", \
    "Authorization": `Bearer ${apiToken}`, \
    "X-GitHub-Api-Version": "2022-11-28", \
    "Content-Type": "application/json", \
    "User-Agent": "Deno-GitHub-Models-Client/1.0", \
  }); \
 \
  const response = await fetch("https://api.github.com/inference/chat/completions", { \
    method: "POST", \
    headers: headers, \
    body: JSON.stringify(requestBody), // The request body must be a JSON string \
  }); \
 \
  if (!response.ok) { \
    // In a real app, parse the error body for more details (see Section 8) \
    throw new Error(`API request failed: ${response.status} ${response.statusText}`); \
  } \
 \
  return await response.json(); \
} \


In this code, the body property of the RequestInit object is set to the result of JSON.stringify(), which serializes the JavaScript request object into the required JSON string format. The Content-Type header is set to application/json to inform the server of the body's format.<sup>6</sup>


### **4.3 Processing the Synchronous JSON Response**

For a successful non-streaming request, the API returns an HTTP 200 OK status and a JSON object containing the model's completion.<sup>3</sup> The structure of this response is predictable and can be strongly typed for easier processing.


    TypeScript

// lib/github_models_api.ts (continued) \
 \
// Interface for the chat completion response \
export interface ChatCompletionResponse { \
  choices: { \
    message: { \
      role: "assistant"; \
      content: string; \
    }; \
    // Other fields like 'finish_reason' might be present \
  }; \
  // Other top-level fields like 'usage' might be present \
} \


The most important part of the response is the choices array. For most standard requests, this array will contain a single element. The model's generated text is located at response.choices.message.content.


### **4.4 Full Deno Example: A Simple Command-Line Chat Client**

This complete, runnable example ties together the concepts from the preceding sections. It creates a simple interactive command-line tool that takes user input, sends it to the GitHub Models API, and prints the assistant's response.


    TypeScript

// main.ts \
import { createChatCompletion, ChatCompletionRequest } from "./lib/github_models_api.ts"; \
 \
// Ensure this file also imports the interfaces from the library file \
 \
const apiToken = Deno.env.get("GITHUB_TOKEN"); \
if (!apiToken) { \
  console.error("Error: GITHUB_TOKEN environment variable not set."); \
  Deno.exit(1); \
} \
 \
async function main() { \
  console.log("Simple Chat Client (type 'exit' to quit)"); \
   \
  // A simple conversation history \
  const messages: { role: string; content: string } = [ \
    { \
      role: "system", \
      content: "You are a helpful assistant.", \
    }, \
  ]; \
 \
  while (true) { \
    const userInput = prompt("You: "); \
    if (userInput === null | \
| userInput.toLowerCase() === "exit") { \
      console.log("Exiting chat."); \
      break; \
    } \
 \
    messages.push({ role: "user", content: userInput }); \
 \
    const requestBody: ChatCompletionRequest = { \
      model: "openai/gpt-4o-mini", // Use a specific, available model \
      messages: messages, \
      temperature: 0.7, \
      max_tokens: 150, \
    }; \
 \
    try { \
      console.log("Assistant:..."); \
      const response = await createChatCompletion(apiToken, requestBody); \
       \
      const assistantResponse = response.choices?.message?.content?.trim(); \
       \
      if (assistantResponse) { \
        console.log(`Assistant: ${assistantResponse}`); \
        messages.push({ role: "assistant", content: assistantResponse }); \
      } else { \
        console.log("Assistant: (No response content)"); \
      } \
 \
    } catch (error) { \
      console.error(`\n[API Error] ${error.message}`); \
    } \
  } \
} \
 \
main(); \


To run this script, save it as main.ts, ensure the library code is in lib/github_models_api.ts, and execute:

deno run --allow-net=api.github.com --allow-env=GITHUB_TOKEN --allow-read --env-file main.ts

(Note: --allow-read is needed for prompt in some Deno versions).


## **Section 5: Mastering Real-Time Interaction: Streaming Chat Completions**

For applications requiring real-time feedback, such as interactive chatbots or live code generation, streaming is the superior approach. By setting stream: true in the inference request, the API will push response tokens to the client as they are generated, dramatically improving perceived performance. This section provides a definitive guide to handling these streams in Deno.


### **5.1 The Paradigm of Streaming: Server-Sent Events (SSE) over a POST Request**

When the stream: true parameter is included in the POST request to the /inference/chat/completions endpoint, the server's response behavior changes fundamentally.<sup>3</sup> Instead of waiting for the entire completion to be generated, the server returns an HTTP

200 OK status immediately and keeps the connection open. It then pushes a series of data chunks over this connection.

This stream of data is formatted according to the **Server-Sent Events (SSE)** protocol.<sup>18</sup> SSE is a web standard designed for servers to push data to clients over a standard HTTP connection. It is simpler than WebSockets as it is unidirectional (server-to-client) and does not require a connection upgrade protocol.<sup>20</sup>

A critical implementation detail arises from this design. The standard web EventSource API, which is the browser-native client for SSE, is restricted to making only GET requests.<sup>19</sup> Since the GitHub Models inference endpoint requires a

POST request to send the prompt payload, **developers cannot use new EventSource() to consume the stream**. Instead, the stream must be processed manually from the body of the fetch response. This is a common and important "gotcha" for developers new to this pattern.

The choice of SSE over WebSockets for a POST-based API is a deliberate architectural decision that favors simplicity and statelessness. WebSockets establish a stateful, bidirectional connection, adding complexity to both client and server implementations. SSE, by contrast, operates over a standard, long-lived HTTP request, making it easier to implement and scale on the backend, especially in modern environments like Deno Deploy that use HTTP/2, which mitigates the historical connection-limit issues of SSE over HTTP/1.1.<sup>18</sup> This design is perfectly suited for the unidirectional "server pushes text to client" use case of an LLM response.


### **5.2 Handling ReadableStream in Deno**

When a fetch request is made in Deno, the response.body property is a ReadableStream.<sup>22</sup> This stream provides chunks of data as they arrive from the network. In the case of the Models API, these chunks are binary data (

Uint8Array) representing the UTF-8 encoded SSE messages.

The standard way to process a ReadableStream is to obtain a reader and consume its chunks in a loop until the stream is closed.<sup>24</sup>


    TypeScript

const response = await fetch(url, options); \
const reader = response.body.getReader(); \
 \
while (true) { \
  const { done, value } = await reader.read(); \
  if (done) { \
    break; // Stream has ended \
  } \
  // 'value' is a Uint8Array chunk \
  processChunk(value); \
} \



### **5.3 Decoding the Stream: From Binary to Text**

Since the raw stream chunks are binary, they must be decoded into text before they can be parsed as SSE messages. While one could use a TextDecoder instance on each chunk, a more elegant and efficient approach is to use the TextDecoderStream. This web-standard API, available in Deno, is a transform stream that can be piped into the data flow.<sup>25</sup> It automatically handles decoding

Uint8Array chunks into strings, correctly managing multi-byte characters that might be split across chunk boundaries.

The most idiomatic way to use it is with the pipeThrough method of the ReadableStream:


    TypeScript

const response = await fetch(url, options); \
const textStream = response.body.pipeThrough(new TextDecoderStream()); \


This single line transforms the binary stream (response.body) into a new ReadableStream (textStream) that yields strings instead of Uint8Arrays, greatly simplifying the subsequent parsing logic.<sup>24</sup>


### **5.4 Parsing the SSE Message Format**

An SSE stream consists of plain text messages. Each message can have fields like event, id, or data. Messages are separated by a double newline (\n\n).<sup>19</sup> For the GitHub Models API (and other OpenAI-compatible APIs), the most important field is

data, which is followed by a JSON object.

A typical sequence of data chunks from the stream might look like this:

data: {"id":"chatcmpl-123","object":"chat.completion.chunk","choices":[{"index":0,"delta":{"role":"assistant"},"finish_reason":null}]} \
 \
data: {"id":"chatcmpl-123","object":"chat.completion.chunk","choices":[{"index":0,"delta":{"content":"Hello"},"finish_reason":null}]} \
 \
data: {"id":"chatcmpl-123","object":"chat.completion.chunk","choices":[{"index":0,"delta":{"content":"!"},"finish_reason":null}]} \
 \
data: \


The parsing logic must:



1. Read from the text stream.
2. Split the incoming text into individual lines.
3. Identify lines that start with data:.
4. Extract the content that follows data:.
5. Handle the special termination message data:.
6. Parse the extracted content as JSON.
7. Access the choices.delta.content field to get the newly generated token(s).


### **5.5 End-to-End Deno Implementation: A Real-Time, Asynchronous Chat Client**

The following complete Deno script demonstrates how to properly implement a streaming chat client. It uses the modern for await...of loop to iterate directly over the ReadableStream, which is cleaner than a manual while loop.


    TypeScript

// main_streaming.ts \
import { ChatCompletionRequest } from "./lib/github_models_api.ts"; // Assuming interfaces are exported \
 \
const apiToken = Deno.env.get("GITHUB_TOKEN"); \
if (!apiToken) {   \
  console.error("Error: GITHUB_TOKEN environment variable not set."); \
  Deno.exit(1); \
} \
 \
async function runStreamingChat() { \
  const headers = new Headers({ \
    "Accept": "application/vnd.github+json", \
    "Authorization": `Bearer ${apiToken}`, \
    "X-GitHub-Api-Version": "2022-11-28", \
    "Content-Type": "application/json", \
    "User-Agent": "Deno-GitHub-Models-Client/1.0", \
  }); \
 \
  const requestBody: ChatCompletionRequest = { \
    model: "openai/gpt-4o-mini", \
    messages:, \
    stream: true, // Enable streaming \
  }; \
 \
  const response = await fetch("https://api.github.com/inference/chat/completions", { \
    method: "POST", \
    headers: headers, \
    body: JSON.stringify(requestBody), \
  }); \
 \
  if (!response.ok) { \
    const errorText = await response.text(); \
    throw new Error(`API Error: ${response.status} ${response.statusText} - ${errorText}`); \
  } \
 \
  if (!response.body) { \
    throw new Error("Response body is null"); \
  } \
 \
  // Pipe the binary stream through a text decoder \
  const textStream = response.body.pipeThrough(new TextDecoderStream()); \
 \
  console.log("Assistant:"); \
  // Use a modern for-await-of loop to process the stream of text chunks \
  for await (const chunk of textStream) { \
    // SSE messages can be split across chunks, so we process by line \
    const lines = chunk.split("\n").filter(line => line.trim()!== ""); \
    for (const line of lines) { \
      if (line.startsWith("data: ")) { \
        const data = line.substring(6); // Remove "data: " prefix \
        if (data === "") { \
          console.log("\n\n"); \
          return; // End of stream \
        } \
        try { \
          const parsed = JSON.parse(data); \
          const content = parsed.choices?.delta?.content; \
          if (content) { \
            Deno.stdout.write(new TextEncoder().encode(content)); \
          } \
        } catch (e) { \
          console.error("\nError parsing JSON chunk:", data, e); \
        } \
      } \
    } \
  } \
} \
 \
runStreamingChat().catch(err => console.error(err)); \


This script correctly initiates a streaming request, decodes the binary response body into a text stream, and processes each SSE message to print the content to the console in real-time, providing the characteristic "typing" effect of a live chatbot.


## **Section 6: Advanced Inference Control and Tool Integration**

Beyond basic chat completions, the GitHub Models API provides a suite of advanced parameters that allow for fine-grained control over model behavior, response format, and capabilities. Mastering these parameters is key to graduating from simple prompts to building sophisticated, reliable, and extensible AI applications.


### **6.1 Fine-Tuning Model Behavior**

These parameters influence the probabilistic nature of the model's output, controlling its creativity and tendency for repetition.



* **temperature vs. top_p**: These two parameters control the randomness of the model's token selection.
    * temperature: A value typically between 0 and 2. Lower values (e.g., 0.2) make the model more deterministic and focused, causing it to pick the most likely next tokens. Higher values (e.g., 1.0) increase randomness, allowing for more creative or diverse outputs.
    * top_p (Nucleus Sampling): A value between 0 and 1. It instructs the model to consider only the smallest set of tokens whose cumulative probability mass exceeds the top_p value. For example, top_p: 0.1 means only tokens comprising the top 10% of probability mass are considered for the next selection.
    * **Best Practice**: It is strongly recommended to alter only one of these parameters at a time, as their effects can interfere with each other.<sup>3</sup>
* **frequency_penalty and presence_penalty**: These parameters help to reduce the model's tendency to repeat itself.
    * frequency_penalty: A value between -2.0 and 2.0. Positive values penalize new tokens based on their existing frequency in the text so far, decreasing the likelihood of the model repeating the same line verbatim.<sup>3</sup>
    * presence_penalty: A value between -2.0 and 2.0. Positive values penalize new tokens based on whether they have appeared in the text so far at all, increasing the model's likelihood to talk about new topics.<sup>3</sup>


### **6.2 Managing Response Generation**

These parameters provide explicit control over the length and termination of the generated response.



* max_tokens: An integer that specifies the maximum number of tokens to generate in the completion. It is a crucial safeguard against overly long or costly responses. The total number of tokens in the prompt plus the max_tokens value cannot exceed the model's total context length.<sup>3</sup>
* stop: An array of up to four strings. The API will stop generating further tokens if it encounters any of these sequences.<sup>6</sup> This is useful for forcing the model to end its output at a specific point, like a newline or a custom marker.
* seed: An integer that, when specified, makes the backend attempt to sample deterministically. Repeated requests with the same seed and other parameters should return the same result. However, determinism is not guaranteed.<sup>3</sup> This is invaluable for testing and debugging.


### **6.3 Structured Outputs: Using response_format**

For applications that require machine-readable output, the response_format parameter is a powerful feature that can compel the model to generate a syntactically correct JSON object.



* response_format: { "type": "json_object" }: When this is included in the request, the model is constrained to generate a string that can be parsed as a valid JSON object.<sup>3</sup> This is extremely useful for tasks like data extraction or classification where the output needs to be consumed programmatically. For this to work effectively, the prompt should include instructions for the model to generate JSON.


### **6.4 Extending Model Capabilities: A Detailed Guide to Implementing tools**

The tools functionality is arguably the most powerful advanced feature, transforming the LLM from a text generator into an reasoning engine that can use external systems. This is the foundation for building AI agents.

The process involves a multi-step conversation:



1. **Client Defines Tools**: The client defines a list of available functions in the tools array of the initial request. Each function object has a name, a description of what it does, and parameters defined as a JSON Schema object.<sup>3</sup> The \
description is critical, as the model uses it to decide when to call the function.
2. **Model Requests a Tool Call**: If the model determines that one of the provided tools can help answer the user's query, it will not respond with a text message. Instead, its response will contain a tool_calls array. Each object in this array specifies the id of the call and the function to be called, including the name and the arguments as a JSON string.
3. **Client Executes the Tool**: The client-side code parses the tool_calls response, identifies the requested function and its arguments, and executes the corresponding local code (e.g., calling a weather API, querying a database).
4. **Client Submits the Result**: The client sends a new request to the model. This request includes the original conversation history, the model's tool_calls response, and a new message with role: "tool". This tool message contains the content (the result of the function execution) and the tool_call_id to link it to the specific request.
5. **Model Summarizes the Result**: The model receives the tool's output and uses that information to generate a final, natural-language response to the user's original query.

The tool_choice parameter can be used to control this behavior, for example, by forcing the model to call a specific function (tool_choice: {"type": "function", "function": {"name": "my_function"}}) or preventing it from calling any (tool_choice: "none").<sup>3</sup>


<table>
  <tr>
   <td>Parameter
   </td>
   <td>Type
   </td>
   <td>Description & Supported Values
   </td>
  </tr>
  <tr>
   <td>frequency_penalty
   </td>
   <td>number
   </td>
   <td>Value between -2.0 and 2.0. Positive values decrease the likelihood of repeating tokens based on frequency.
   </td>
  </tr>
  <tr>
   <td>presence_penalty
   </td>
   <td>number
   </td>
   <td>Value between -2.0 and 2.0. Positive values decrease the likelihood of repeating tokens based on presence.
   </td>
  </tr>
  <tr>
   <td>response_format
   </td>
   <td>object
   </td>
   <td>Specifies the output format. Example: { "type": "json_object" }.
   </td>
  </tr>
  <tr>
   <td>seed
   </td>
   <td>integer
   </td>
   <td>If specified, attempts to produce deterministic output. Not guaranteed.
   </td>
  </tr>
  <tr>
   <td>stop
   </td>
   <td>array of strings
   </td>
   <td>Up to 4 sequences where the API will stop generating further tokens.
   </td>
  </tr>
  <tr>
   <td>tool_choice
   </td>
   <td>string or object
   </td>
   <td>Controls if/how the model uses tools. Can be none, auto, required, or an object specifying a function to call.
   </td>
  </tr>
  <tr>
   <td>tools
   </td>
   <td>array of objects
   </td>
   <td>A list of tools (functions) the model may call. Each tool has a name, description, and parameters (JSON Schema).
   </td>
  </tr>
  <tr>
   <td>top_p
   </td>
   <td>number
   </td>
   <td>Value between 0 and 1. Nucleus sampling alternative to temperature.
   </td>
  </tr>
</table>


Table 6.1: Advanced Inference Request Body Parameters. Data sourced from.<sup>3</sup>


## **Section 7: Organizational Context: Attributing API Usage**

For developers working within a corporate or team environment on GitHub, it is often necessary to attribute API usage to their specific organization for billing, tracking, and governance purposes. The GitHub Models API provides a dedicated endpoint for this exact scenario.


### **7.1 Utilizing the POST /orgs/{org}/inference/chat/completions Endpoint**

This specialized endpoint mirrors the functionality of the standard inference endpoint but includes an organization identifier in the URL path.



* **Endpoint**: POST https://api.github.com/orgs/{org}/inference/chat/completions <sup>3</sup>

Here, {org} must be replaced with the login name of the target GitHub Organization (e.g., github). The request body, headers, and response format are identical to the non-organizational endpoint described in previous sections.<sup>3</sup> All parameters, including

model, messages, stream, and advanced controls, function in exactly the same way. The sole purpose of this endpoint is to link the inference request to the specified organization.<sup>7</sup>


### **7.2 Prerequisites and Permissions for Organizational Attribution**

To successfully use this endpoint, two conditions must be met:



1. **Organization Membership**: The user whose Personal Access Token is used for authentication must be a member of the organization specified in the {org} path parameter.<sup>3</sup>
2. **Models Enabled**: The target organization must have the GitHub Models feature enabled in its settings. If the feature is not enabled for the organization, requests to this endpoint will likely fail.<sup>3</sup>

The authentication token must still possess the models:read permission, as this is the foundational requirement for any interaction with the Models API.<sup>3</sup>


### **7.3 Comparative Example: Adapting the Deno Client**

Modifying the Deno client from previous sections to use the organizational endpoint is a straightforward change. It primarily involves constructing a different URL string.


    TypeScript

// lib/github_models_api.ts (adding a new function) \
 \
/** \
 * Performs a chat completion request attributed to a specific organization. \
 * @param apiToken A fine-grained GitHub PAT with 'models:read' permission. \
 * @param org The login of the GitHub organization. \
 * @param requestBody The body of the chat completion request. \
 * @returns A promise that resolves to the API response object. \
 */ \
export async function createOrgChatCompletion( \
  apiToken: string, \
  org: string, \
  requestBody: ChatCompletionRequest \
) { \
  const headers = new Headers({ \
    "Accept": "application/vnd.github+json", \
    "Authorization": `Bearer ${apiToken}`, \
    "X-GitHub-Api-Version": "2022-11-28", \
    "Content-Type": "application/json", \
    "User-Agent": "Deno-GitHub-Models-Client/1.0", \
  }); \
 \
  // The only change is the URL construction \
  const apiUrl = `https://api.github.com/orgs/${org}/inference/chat/completions`; \
 \
  const response = await fetch(apiUrl, { \
    method: "POST", \
    headers: headers, \
    body: JSON.stringify(requestBody), \
  }); \
 \
  if (!response.ok) { \
    throw new Error(`API request failed: ${response.status} ${response.statusText}`); \
  } \
 \
  return await response.json(); \
} \


This demonstrates that integrating organizational attribution into an existing application is a minor code change, provided the necessary permissions and settings are in place.


## **Section 8: A Researcher's Guide to Error Handling and Diagnostics**

A production-quality application must be resilient to failure. Interacting with a remote API introduces potential points of failure, from network issues to invalid inputs. This section provides a systematic methodology for diagnosing and handling errors when working with the GitHub Models API.


### **8.1 A Taxonomy of Common HTTP Status Codes**

Understanding the meaning of different HTTP status codes returned by the API is the first step in effective error handling.


<table>
  <tr>
   <td>HTTP Status
   </td>
   <td>Likely Cause
   </td>
   <td>Resolution/Action
   </td>
  </tr>
  <tr>
   <td>401 Unauthorized
   </td>
   <td>The Authorization header is missing, or the provided token is invalid or expired.
   </td>
   <td>Verify the token is correct, not expired, and included in the request as Bearer &lt;TOKEN>. Regenerate the token if necessary.
   </td>
  </tr>
  <tr>
   <td>403 Forbidden
   </td>
   <td>The token is valid but lacks the required models:read permission. Can also indicate that a primary or secondary rate limit has been exceeded.
   </td>
   <td>Ensure the fine-grained PAT has the models:read permission. If it's a rate limit error, inspect headers and implement backoff.
   </td>
  </tr>
  <tr>
   <td>404 Not Found
   </td>
   <td>The endpoint URL is incorrect (e.g., a typo, a trailing slash). Can also occur when trying to access a resource tied to a private context (like an org) without sufficient permissions.
   </td>
   <td>Double-check the request URL against the documentation. Verify authentication and permissions if the resource should exist.
   </td>
  </tr>
  <tr>
   <td>422 Unprocessable Entity
   </td>
   <td>The request body is syntactically valid JSON, but it fails the API's semantic validation. Examples: an invalid model ID, an unsupported combination of parameters (e.g., modalities), or a malformed messages array.
   </td>
   <td><strong>Parse the JSON error body.</strong> The response contains detailed information about which field is invalid. Correct the request payload based on the error details.
   </td>
  </tr>
  <tr>
   <td>429 Too Many Requests
   </td>
   <td>The client has exceeded a specific rate limit.
   </td>
   <td>Stop making requests. Check for a Retry-After header and wait for that duration. Otherwise, implement an exponential backoff strategy.
   </td>
  </tr>
</table>


Table 8.1: Common API Error Responses and Resolutions. Data sourced from.<sup>12</sup>

GitHub intentionally uses a 404 Not Found response instead of 403 Forbidden when an authenticated user attempts to access a private resource they do not have permission for. This is a security measure to avoid leaking information about the existence of private repositories or organizations.<sup>28</sup>


### **8.2 Diagnosing 422 Unprocessable Entity: Deconstructing the Validation Error**

The 422 Unprocessable Entity error is one of the most common issues developers face when first integrating with an API. It signifies that the server understood the request, but the data provided in the payload is invalid in some way.<sup>30</sup> The key to resolving this error is to inspect the response body, which contains a structured JSON object detailing the failure.

For instance, a 422 error from a GitHub API might return a body like this <sup>31</sup>:


    JSON

{ \
  "message": "Validation Failed", \
  "errors": [ \
    { \
      "resource": "ChatCompletion", \
      "field": "model", \
      "code": "missing" \
    } \
  ], \
  "documentation_url": "https://docs.github.com/en/rest/models/inference" \
} \


In another case, an unsupported combination of modalities in the request will result in a 422 error.<sup>3</sup> Similarly, issues with JSON deserialization, such as providing an array where a string is expected in the

messages content, can trigger this error.<sup>33</sup>

A robust error handler must not just check for the 422 status code; it must attempt to parse the response body as JSON and extract the message and errors fields to provide a clear, actionable diagnostic message to the developer or user.


### **8.3 Navigating Rate Limits: Proactive and Reactive Strategies**

The GitHub API enforces rate limits to ensure service stability for all users.<sup>8</sup> Exceeding these limits will result in a

403 Forbidden or 429 Too Many Requests response. To handle this gracefully, a client should use the informational headers returned with every API response:



* x-ratelimit-remaining: The number of requests remaining in the current window.
* x-ratelimit-reset: The time at which the current rate limit window resets, in UTC epoch seconds.
* retry-after: If present in a 429 or 403 response, this header indicates the number of seconds the client should wait before making another request.

A reactive strategy involves implementing an exponential backoff algorithm. When a rate limit error is received, the client should wait for the duration specified by retry-after, or if that is not present, wait for an exponentially increasing amount of time between retries (e.g., 1s, 2s, 4s, 8s) before attempting the request again.<sup>34</sup>


### **8.4 A Robust Error Handling Wrapper for the Deno fetch Client**

To centralize and simplify error handling, it is best practice to create a wrapper function around the fetch call. This function can standardize the process of checking the response and throwing a detailed, custom error.


    TypeScript

// lib/error_handler.ts \
 \
export class ApiError extends Error { \
  constructor( \
    public status: number, \
    public statusText: string, \
    public errorBody?: any \
  ) { \
    const detail = errorBody? JSON.stringify(errorBody, null, 2) : 'No error body'; \
    super(`API Error ${status} ${statusText}:\n${detail}`); \
    this.name = "ApiError"; \
  } \
} \
 \
export async function fetchWithHandling(url: string | URL, options: RequestInit) { \
  const response = await fetch(url, options); \
 \
  if (!response.ok) { \
    let errorBody; \
    try { \
      // Attempt to parse the error response as JSON \
      errorBody = await response.json(); \
    } catch { \
      // If parsing fails, the body might be text or empty \
      errorBody = await response.text().catch(() => 'Could not read error body.'); \
    } \
    throw new ApiError(response.status, response.statusText, errorBody); \
  } \
 \
  return response; \
} \
 \
// Example usage in another function \
/* \
import { fetchWithHandling, ApiError } from "./lib/error_handler.ts"; \
try { \
  const response = await fetchWithHandling(apiUrl, options); \
  //... process successful response \
} catch (error) { \
  if (error instanceof ApiError) { \
    console.error(`Caught an API Error: Status ${error.status}`); \
    // Now you can inspect error.errorBody for details \
    if (error.status === 422) { \
      console.error("Validation failed:", error.errorBody?.errors); \
    } \
  } else { \
    console.error("An unexpected error occurred:", error); \
  } \
} \
*/ \


This wrapper provides a clean try...catch pattern for all API calls, ensuring that both expected API errors and unexpected network failures are handled gracefully and informatively.


## **Section 9: Best Practices and Performance Considerations**

Adhering to established best practices ensures that an application interacting with the GitHub Models API is efficient, resilient, and a good citizen of the API ecosystem. This final section provides architectural recommendations and performance considerations.


### **9.1 Efficient Stream Consumption Patterns**

When working with streaming responses, the choice of consumption pattern can impact code clarity and maintainability.



* **Prefer for await...of Loops**: For consuming ReadableStream objects in Deno, the for await...of loop is the modern and recommended approach. It is more declarative and less prone to boilerplate errors than a manual while (true) loop with a reader.read() call.
* **Handle Stream Cancellation**: If the consumer of the data (e.g., a user closing a chat window) no longer needs the stream, the underlying fetch request should be cancelled to free up resources on both the client and server. This can be achieved by passing an AbortSignal from an AbortController to the fetch options and calling controller.abort() when necessary.


### **9.2 Strategies for Token Management**

The security of the API token is paramount.



* **Use Environment Variables**: As detailed in Section 2, API tokens should always be loaded from environment variables, never hardcoded. Deno's --allow-env flag provides a secure way to grant access to only the necessary variables.<sup>14</sup>
* **Production Secret Management**: For production deployments, especially on platforms like Deno Deploy, use the platform's built-in secret management features. These systems store secrets encrypted at rest and inject them securely into the runtime environment, which is a more robust solution than relying on .env files in a production context.<sup>16</sup>


### **9.3 API Best Practices from GitHub**

GitHub provides official best practices for using its REST API, which are fully applicable to the Models API endpoints.<sup>34</sup>



* **Provide a Meaningful User-Agent**: All API requests should include a User-Agent header that identifies the application. This allows GitHub to contact the developer if a script is causing problems or to gather statistics on API usage. By default, many clients send a generic agent, but a custom one is preferred.<sup>11</sup> \
TypeScript \
const headers = new Headers({ \
  "User-Agent": "My-AI-App/1.0 (https://myapp.com/info)" \
}); \

* **Follow HTTP Redirects**: The API may use HTTP redirection (301, 302, 307 status codes) to forward a request to a new URL. Deno's fetch API handles these redirects automatically by default, but developers should be aware that this behavior exists and is not an error.<sup>34</sup>
* **Do Not Manually Parse URLs**: API responses may contain fields with full URLs. Developers should always use these URLs directly rather than attempting to manually construct or parse them. This prevents the application from breaking if GitHub changes its URL structures in the future.<sup>34</sup>
* **Pause Between Requests**: While the Models API is primarily for reading data, making a large number of requests in a tight, concurrent loop can trigger secondary rate limits designed to prevent abuse. It is good practice to make requests serially or to introduce a small delay between them if making them in large batches.<sup>34</sup>

By integrating these best practices into the application architecture, developers can build robust, scalable, and secure solutions on top of the powerful capabilities offered by the GitHub Models API.


#### Works cited



1. Models - GitHub Docs, accessed June 18, 2025, [https://docs.github.com/en/rest/models](https://docs.github.com/en/rest/models)
2. REST API endpoints for models catalog - GitHub Docs, accessed June 18, 2025, [https://docs.github.com/en/rest/models/catalog](https://docs.github.com/en/rest/models/catalog)
3. REST API endpoints for models inference - GitHub Docs, accessed June 18, 2025, [https://docs.github.com/en/rest/models/inference](https://docs.github.com/en/rest/models/inference)
4. Prototyping with AI models - GitHub Docs, accessed June 18, 2025, [https://docs.github.com/en/github-models/use-github-models/prototyping-with-ai-models](https://docs.github.com/en/github-models/use-github-models/prototyping-with-ai-models)
5. GitHub REST API documentation, accessed June 18, 2025, [https://docs.github.com/en/rest](https://docs.github.com/en/rest)
6. REST API endpoints for models inference - Documentación de GitHub, accessed June 18, 2025, [https://docs.github.com/es/rest/models/inference](https://docs.github.com/es/rest/models/inference)
7. GitHub Models API now available - GitHub Changelog, accessed June 18, 2025, [https://github.blog/changelog/2025-05-15-github-models-api-now-available/](https://github.blog/changelog/2025-05-15-github-models-api-now-available/)
8. Prototyping with AI models - GitHub Docs, accessed June 18, 2025, [https://docs.github.com/github-models/prototyping-with-ai-models](https://docs.github.com/github-models/prototyping-with-ai-models)
9. REST API endpoints for organizations - GitHub Docs, accessed June 18, 2025, [https://docs.github.com/en/rest/orgs](https://docs.github.com/en/rest/orgs)
10. REST API endpoints for pull requests - GitHub Docs, accessed June 18, 2025, [https://docs.github.com/en/rest/pulls/pulls](https://docs.github.com/en/rest/pulls/pulls)
11. Getting started with the REST API - GitHub Docs, accessed June 18, 2025, [https://docs.github.com/rest/guides/getting-started-with-the-rest-api](https://docs.github.com/rest/guides/getting-started-with-the-rest-api)
12. Authenticating to the REST API - GitHub Docs, accessed June 18, 2025, [https://docs.github.com/en/rest/authentication/authenticating-to-the-rest-api](https://docs.github.com/en/rest/authentication/authenticating-to-the-rest-api)
13. Quickstart for GitHub REST API, accessed June 18, 2025, [https://docs.github.com/en/rest/quickstart](https://docs.github.com/en/rest/quickstart)
14. Environment variables - Deno Docs, accessed June 18, 2025, [https://docs.deno.com/examples/environment_variables/](https://docs.deno.com/examples/environment_variables/)
15. Environment variables - Deno Docs, accessed June 18, 2025, [https://docs.deno.com/runtime/reference/env_variables/](https://docs.deno.com/runtime/reference/env_variables/)
16. Environment Variables and Contexts - Deno Docs, accessed June 18, 2025, [https://docs.deno.com/deploy/early-access/reference/env-vars-and-contexts/](https://docs.deno.com/deploy/early-access/reference/env-vars-and-contexts/)
17. Environment variables - Val Town Docs, accessed June 18, 2025, [https://docs.val.town/reference/environment-variables/](https://docs.val.town/reference/environment-variables/)
18. I made a library that makes it simple to use server-sent events: real-time server-to-client communication without WebSockets : r/Deno - Reddit, accessed June 18, 2025, [https://www.reddit.com/r/Deno/comments/1kvuibq/i_made_a_library_that_makes_it_simple_to_use/](https://www.reddit.com/r/Deno/comments/1kvuibq/i_made_a_library_that_makes_it_simple_to_use/)
19. Using server-sent events - Web APIs | MDN, accessed June 18, 2025, [https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events)
20. Web Streams at the Edge - Deno, accessed June 18, 2025, [https://deno.com/blog/deploy-streams](https://deno.com/blog/deploy-streams)
21. lukeed/fetch-event-stream: A tiny (736b) utility for Server Sent Event (SSE) streaming via `fetch` and Web Streams API - GitHub, accessed June 18, 2025, [https://github.com/lukeed/fetch-event-stream](https://github.com/lukeed/fetch-event-stream)
22. Fetch and stream data - Deno Docs, accessed June 18, 2025, [https://docs.deno.com/examples/fetch_data_tutorial/](https://docs.deno.com/examples/fetch_data_tutorial/)
23. ReadableStream - Streams - Web documentation - Deno Docs, accessed June 18, 2025, [https://docs.deno.com/api/web/~/ReadableStream](https://docs.deno.com/api/web/~/ReadableStream)
24. Streaming requests with the fetch API | Capabilities - Chrome for Developers, accessed June 18, 2025, [https://developer.chrome.com/docs/capabilities/web-apis/fetch-streaming-requests](https://developer.chrome.com/docs/capabilities/web-apis/fetch-streaming-requests)
25. TextDecoderStream - stream/web - Node documentation - Deno Docs, accessed June 18, 2025, [https://docs.deno.com/api/node/stream/web/~/TextDecoderStream](https://docs.deno.com/api/node/stream/web/~/TextDecoderStream)
26. TextDecoderStream - Web APIs | MDN, accessed June 18, 2025, [https://developer.mozilla.org/en-US/docs/Web/API/TextDecoderStream](https://developer.mozilla.org/en-US/docs/Web/API/TextDecoderStream)
27. TextDecoderStream - Encoding - Web documentation - Deno Docs, accessed June 18, 2025, [https://docs.deno.com/api/web/~/TextDecoderStream](https://docs.deno.com/api/web/~/TextDecoderStream)
28. Troubleshooting the REST API - GitHub Docs, accessed June 18, 2025, [https://docs.github.com/en/rest/using-the-rest-api/troubleshooting-the-rest-api](https://docs.github.com/en/rest/using-the-rest-api/troubleshooting-the-rest-api)
29. landonline-api-restful-guidelines/chapters/http-status-codes-and-errors.adoc at main, accessed June 18, 2025, [https://github.com/linz/landonline-api-restful-guidelines/blob/main/chapters/http-status-codes-and-errors.adoc](https://github.com/linz/landonline-api-restful-guidelines/blob/main/chapters/http-status-codes-and-errors.adoc)
30. 422 Unprocessable Entity when I PUT data on repository #55336 - GitHub, accessed June 18, 2025, [https://github.com/orgs/community/discussions/55336](https://github.com/orgs/community/discussions/55336)
31. Unexpected error response from GitHub API 422 when attempting to create issue, accessed June 18, 2025, [https://stackoverflow.com/questions/54008483/unexpected-error-response-from-github-api-422-when-attempting-to-create-issue](https://stackoverflow.com/questions/54008483/unexpected-error-response-from-github-api-422-when-attempting-to-create-issue)
32. REST API endpoints for issues - GitHub Docs, accessed June 18, 2025, [https://docs.github.com/en/rest/issues/issues](https://docs.github.com/en/rest/issues/issues)
33. DeepSeek API Error: 422 Failed to deserialize the JSON · Issue #230 - GitHub, accessed June 18, 2025, [https://github.com/saoudrizwan/claude-dev/issues/230](https://github.com/saoudrizwan/claude-dev/issues/230)
34. Best practices for using the REST API - GitHub Docs, accessed June 18, 2025, [https://docs.github.com/en/rest/using-the-rest-api/best-practices-for-using-the-rest-api](https://docs.github.com/en/rest/using-the-rest-api/best-practices-for-using-the-rest-api)
35. Environment variables - Deno Docs, accessed June 18, 2025, [https://docs.deno.com/deploy/manual/environment-variables/](https://docs.deno.com/deploy/manual/environment-variables/)
