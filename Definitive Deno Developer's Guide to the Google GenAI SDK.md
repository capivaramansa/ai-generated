

# **Definitive Deno Developer's Guide to the @google/genai SDK**

## **Introduction: The New Era of Google's Generative AI SDK**

The landscape of generative artificial intelligence is characterized by rapid evolution, not only in the capabilities of the models themselves but also in the tools developers use to harness them. In a significant step towards unifying and simplifying this developer experience, Google has introduced the **@google/genai** SDK for TypeScript and JavaScript. This new library represents a fundamental shift, consolidating access to Google's state-of-the-art generative models, including the Gemini family, Imagen, and Veo, under a single, cohesive interface.

### **The Unified Vision**

The @google/genai SDK is engineered with a unified vision: to provide a single, consistent pathway for developers to integrate generative AI into their applications, regardless of the deployment target. It supports both the **Gemini Developer API**, accessible via API keys from Google AI Studio for rapid prototyping, and the enterprise-grade **Vertex AI platform** for scalable, production-ready solutions. This dual-backend support is a cornerstone of the SDK's design, enabling a seamless development lifecycle where an application can be prototyped using the Gemini Developer API and later migrated to Vertex AI without requiring significant code rewrites.

### **Why the Change? A Developer-Centric Approach**

The introduction of @google/genai is more than a mere package rename; it is a re-architecture born from extensive developer feedback on its predecessor, @google/generative-ai, and inspired by the best practices observed across the broader ecosystem of AI SDKs. The primary objective was to create an "extremely simple and clear path for developers to build with our models".

This developer-centric philosophy manifests in a more modular and explicit API design. Where the older SDK often overloaded a single GenerativeModel object with disparate responsibilities, the new SDK organizes functionality into logical, self-contained submodules like ai.models, ai.chats, and ai.files. This separation of concerns enhances code readability, maintainability, and discoverability, allowing developers to interact with the API in a more intuitive and structured manner.

### **A Glimpse into the Future**

Adopting the @google/genai SDK is not just about embracing a new API; it is about future-proofing applications. Google has designated this SDK as the "vanilla" offering where all new features and models will be introduced first. This includes cutting-edge capabilities such as the **Live API** for real-time, interactive multimodal sessions, advanced **Multi-modal Coherent Prompting (MCP)**, and access to new foundational models for image and video generation. Consequently, the feature gap between the new and the deprecated SDK is expected to widen over time, making migration a strategic imperative for any project that aims to stay at the forefront of generative AI.

### **Deno as a Premier Platform**

While much of the official documentation and examples for @google/genai are presented in a Node.js context, the Deno runtime emerges as a particularly well-suited environment for leveraging this new SDK. Deno's modern architecture, which features first-class TypeScript and JSX support, a comprehensive built-in toolchain, and a security-by-default model, aligns remarkably well with the goals of modern application development. Its native support for the npm: specifier provides a robust and seamless bridge to the vast npm ecosystem, allowing Deno developers to use packages like @google/genai without friction.

The architectural philosophy of the @google/genai SDK—with its explicit, modular structure—finds a natural synergy with Deno's own design principles. Deno encourages developers to be explicit about their dependencies and the permissions their code requires. The new SDK's clear separation of concerns into submodules like ai.models, ai.files, and ai.chats mirrors this philosophy, creating a development experience that feels native and intuitive within the Deno ecosystem. This guide is dedicated to exploring this synergy, providing a comprehensive, Deno-native roadmap for building powerful AI applications with Google's next-generation models.

## **Chapter 1: Getting Started in a Deno Environment**

This chapter provides a comprehensive guide to setting up a Deno project, managing API keys securely, and making your first successful API call to the Gemini API using the @google/genai SDK.

### **1.1 Prerequisites: Setting Up Your Deno Project**

Before interacting with the SDK, a proper Deno project structure is essential. Deno's built-in tooling simplifies this process significantly.

First, ensure the Deno command-line interface (CLI) is installed on your system. If not, it can be installed using one of the commands provided on the official Deno website.

With the Deno CLI available, create a new project directory and initialize it. This command creates a standard project scaffold, including main.ts and deno.json files, which are central to managing your application and its dependencies.

Bash

mkdir my-gemini-app  
cd my-gemini-app  
deno init

The deno.json file will serve as the configuration hub for your project, where you will define import maps and tasks. The main.ts file will be the entry point for your application code.

### **1.2 Secure API Key Management in Deno**

All interactions with the Gemini API require authentication, typically via an API key. It is of paramount importance to handle these keys securely.

1\. Obtain an API Key  
You can acquire a free API key from Google AI Studio. This key is used to authenticate requests to the Gemini Developer API.  
2\. Security Best Practices  
A critical security principle is to never expose API keys in client-side code or commit them directly into version control. The recommended approach for server-side applications, including those built with Deno, is to manage keys using environment variables.  
3\. Using Environment Variables in Deno  
The idiomatic way to manage environment variables in a Deno project is by using a .env file.

1. Create a file named .env in the root of your project.  
2. Add your API key to this file:  
   \#.env  
   GEMINI\_API\_KEY="YOUR\_API\_KEY\_HERE"

3. To load these variables into your Deno application's environment, you can use the load function from Deno's standard library for dotenv. Create a \_cli.ts or main.ts file to handle this loading before your main application logic runs.  
   TypeScript  
   // \_cli.ts  
   import { load } from "https://deno.land/std@0.224.0/dotenv/mod.ts";

   // Load environment variables from.env file  
   await load({ export: true });

   // Now, import and run your main application logic  
   await import("./main.ts");

4. Within your application code (main.ts), you can now securely access the API key using Deno.env.get(). This is the Deno equivalent of process.env in Node.js.  
   TypeScript  
   // main.ts  
   const apiKey \= Deno.env.get("GEMINI\_API\_KEY");  
   if (\!apiKey) {  
     throw new Error("GEMINI\_API\_KEY environment variable not set.");  
   }

### **1.3 Importing and Initializing the SDK**

Deno's native support for npm packages via the npm: specifier makes integrating the @google/genai SDK straightforward.

Method 1: Direct Import  
You can import the package directly within any TypeScript file. It is a best practice to pin the package version to ensure stable and reproducible builds.

TypeScript

import { GoogleGenAI } from "npm:@google/genai@^1.5.1";

Method 2 (Recommended): Using deno.json Import Maps  
The most idiomatic and maintainable approach in Deno is to use an import map within your deno.json file. This creates a clean, project-wide alias for the package, making your import statements shorter and centralizing dependency management.

1. Edit your deno.json file:  
   JSON  
   // deno.json  
   {  
     "imports": {  
       "@google/genai": "npm:@google/genai@^1.5.1"  
     }  
   }

2. Now, you can use the clean alias in your code:  
   TypeScript  
   // main.ts  
   import { GoogleGenAI } from "@google/genai";

### **1.4 Your First API Call: "Hello, Gemini\!"**

Let's combine these steps into a complete, runnable Deno script that sends a prompt to the Gemini model and prints the response.

1. Ensure your .env file is created and populated.  
2. Configure your deno.json with the import map as shown above.  
3. Create the following main.ts file:  
   TypeScript  
   // main.ts  
   import { GoogleGenAI } from "@google/genai";

   // Get the API key from the environment variables.  
   // Deno.env.get() is the secure way to access them.  
   const apiKey \= Deno.env.get("GEMINI\_API\_KEY");  
   if (\!apiKey) {  
     throw new Error("GEMINI\_API\_KEY environment variable not set.");  
   }

   // Initialize the GoogleGenAI client with the API key.  
   const ai \= new GoogleGenAI({ apiKey });

   async function run() {  
     // For text-only input, use the gemini-2.0-flash model  
     const model \= "gemini-2.0-flash";  
     const prompt \= "Write a short, engaging story about a programmer who discovers a secret in Deno's source code.";

     console.log(\`Sending prompt to ${model}...\`);

     // The core generation call  
     const response \= await ai.models.generateContent({  
       model: model,  
       contents: prompt,  
     });

     // The.text accessor provides a convenient way to get the full text response.  
     const responseText \= response.text;  
     console.log("\\n--- Gemini's Response \---");  
     console.log(responseText);  
     console.log("-------------------------\\n");  
   }

   run().catch(console.error);

4. To execute this script, you must grant the necessary permissions to the Deno runtime. In this case, it needs access to environment variables (--allow-env) and the network to make the API call (--allow-net).  
   Bash  
   \# First, load the.env file and then run the main script  
   deno run \--allow-env \--allow-net main.ts

This command will execute your script, securely connect to the Gemini API, and print the model's generated story to your console, confirming that your Deno environment is correctly configured to use the @google/genai SDK.

## **Chapter 2: Migrating from @google/generative-ai**

For developers with existing projects built on the deprecated @google/generative-ai SDK, transitioning to the new @google/genai package is a crucial step to access the latest features and ensure long-term support. This chapter provides a clear migration path, highlighting the key architectural changes and offering a definitive cheat sheet for updating your code in a Deno environment.

### **2.1 Understanding the Architectural Shift**

The migration from the old SDK to the new one involves more than just changing package names; it reflects a fundamental shift in the API's design philosophy. Understanding this shift is key to writing idiomatic code with the new SDK.

* **From Model-Centric to Client-Centric:** The old SDK operated on a **model-centric** pattern. A developer would first instantiate a specific GenerativeModel object, configured with a model name and generation settings. All subsequent actions, like generating content or starting a chat, were methods called on this model instance.  
* **The New Paradigm:** The @google/genai SDK introduces a **client-centric** architecture. A developer now creates a single, unified GoogleGenAI client instance. This client acts as a central hub, organizing API features into distinct, logical submodules such as ai.models, ai.chats, and ai.files. The model name is no longer tied to an object instance but is passed as a parameter in each request.

This architectural change offers several advantages:

1. **Separation of Concerns:** Functionalities are cleanly separated. If you are working with conversational state, you use ai.chats. If you are managing files, you use ai.files. This makes the API surface more intuitive and the code more self-documenting.  
2. **Flexibility:** Since the model is specified per-request, a single client instance can easily interact with multiple different models (gemini-2.0-flash, gemini-2.5-pro, etc.) without needing to create multiple model objects.  
3. **Consistency:** The pattern of client.submodule.method({...request }) is consistent across the entire SDK, making it easier to learn and use.

### **2.2 The Definitive Migration Cheat Sheet for Deno**

This table serves as a quick-reference guide for migrating common patterns from @google/generative-ai to @google/genai, with all examples specifically adapted for the Deno runtime.

| Use Case | Old SDK (@google/generative-ai) in Deno | New SDK (@google/genai) in Deno | Key Change & Rationale |
| :---- | :---- | :---- | :---- |
| **Installation & Import** | import { GoogleGenerativeAI } from "npm:@google/generative-ai"; | import { GoogleGenAI } from "npm:@google/genai"; | Package rename from @google/generative-ai to @google/genai. The main class is now GoogleGenAI. |
| **Initialization** | const genAI \= new GoogleGenerativeAI(Deno.env.get("GEMINI\_API\_KEY")\!); | const ai \= new GoogleGenAI({ apiKey: Deno.env.get("GEMINI\_API\_KEY")\! }); | Initialization now uses a configuration object. This is more extensible, allowing for additional options like vertexai: true for Vertex AI integration.1 |
| **Text Generation** | const model \= genAI.getGenerativeModel({ model: "gemini-1.5-flash" }); await model.generateContent("Why is the sky blue?"); | await ai.models.generateContent({ model: "gemini-2.0-flash", contents: "Why is the sky blue?", }); | The action moves from a model instance to the ai.models submodule. The model is now a parameter within the request object, decoupling the action from a specific model instance.1 |
| **Streaming Response** | const result \= await model.generateContentStream("..."); for await (const chunk of result.stream) {... } | const response \= await ai.models.generateContentStream({... }); for await (const chunk of response) {... } | The new SDK returns an async iterable directly from the method call, simplifying the loop. The older SDK required accessing a .stream property on the result object.1 |
| **Chat Session** | const chat \= model.startChat({ history: \[...\] }); await chat.sendMessage("Hello"); | const chat \= ai.chats.create({ model: "gemini-2.0-flash", history: \[...\] }); await chat.sendMessage({ message: "Hello" }); | Chat functionality is now explicitly managed by the ai.chats submodule. This separates conversational state management from the core generation model, leading to cleaner code.1 |
| **Multimodal (Image)** | // Relied on Node.js 'fs' and 'Buffer' function fileToGenerativePart(path, mimeType) { return { inlineData: { data: Buffer.from(fs.readFileSync(path)).toString("base64"), mimeType } }; } | // Uses Deno/Web standards const imageBytes \= await Deno.readFile("image.png"); const imageBase64 \= btoa(String.fromCharCode(...imageBytes)); const imagePart \= { inlineData: { data: imageBase64, mimeType: 'image/png' } }; | The old SDK's examples were Node-centric. The new SDK uses a standard inlineData object that works with Base64 strings, which can be generated using Deno's built-in Deno.readFile and the web-standard btoa function. For larger files, the ai.files.upload method is the recommended, runtime-agnostic approach. |
| **Request Configuration** | const model \= genAI.getGenerativeModel({ model: "gemini-1.5-flash", generationConfig: { temperature: 0.9 } }); | await ai.models.generateContent({ model: "gemini-2.0-flash", contents: "...", config: { temperature: 0.9 } }); | Generation parameters (like temperature) are now passed in a config object within the specific request. This allows for per-call configuration, offering more granular control than setting it on the model instance.1 |
| **Safety Settings** | const model \= genAI.getGenerativeModel({ model: "gemini-1.5-flash", safetySettings: \[...\] }); | await ai.models.generateContent({ model: "gemini-2.0-flash", contents: "...", config: { safetySettings: \[...\] } }); | Similar to generationConfig, safety settings are now part of the per-request config object, providing dynamic control over content filtering for each individual API call.1 |

## **Chapter 3: Deep Dive into the GoogleGenAI Client**

The GoogleGenAI class is the central entry point to the entire SDK. A single instance of this client is typically all that is needed for an application. This chapter explores its initialization for different backends and provides a conceptual overview of its powerful submodules.

### **3.1 The Unified Client: Gemini vs. Vertex AI**

One of the most powerful features of the @google/genai SDK is its ability to seamlessly target two distinct Google Cloud backends: the Gemini Developer API and the Vertex AI platform. This allows projects to evolve from simple prototypes to enterprise-scale applications with minimal code changes. The backend is selected during the initialization of the GoogleGenAI client.

#### **Gemini Developer API Initialization**

For rapid development, experimentation, and smaller-scale applications, the Gemini Developer API is the ideal starting point. It uses a simple API key for authentication, which can be obtained from Google AI Studio.

Initialization is straightforward, requiring only the apiKey to be passed in the configuration object.

TypeScript

// Deno example for Gemini Developer API  
import { GoogleGenAI } from "npm:@google/genai";

const apiKey \= Deno.env.get("GEMINI\_API\_KEY");  
if (\!apiKey) {  
  throw new Error("GEMINI\_API\_KEY environment variable is not set.");  
}

// Initialize the client for the Gemini Developer API  
const ai \= new GoogleGenAI({ apiKey });

console.log("Client initialized for Gemini Developer API.");

#### **Vertex AI Initialization**

When an application needs to scale or requires enterprise-grade features like fine-tuning, robust data governance, and integration with other Google Cloud services, migrating to Vertex AI is the next logical step. The SDK makes this transition simple.

To initialize the client for Vertex AI, you must provide three key pieces of information:

1. vertexai: true: A boolean flag that tells the SDK to target the Vertex AI backend.  
2. project: Your Google Cloud Project ID.  
3. location: The Google Cloud region where your project is hosted (e.g., us-central1).

Authentication for Vertex AI in a local Deno development environment is typically handled via **Application Default Credentials (ADC)**. You must first authenticate using the Google Cloud CLI:

Bash

gcloud auth application-default login

Once authenticated, the SDK will automatically pick up your credentials. The Deno code for initialization is as follows:

TypeScript

// Deno example for Vertex AI  
import { GoogleGenAI } from "npm:@google/genai";

const projectId \= Deno.env.get("GOOGLE\_CLOUD\_PROJECT");  
const location \= Deno.env.get("GOOGLE\_CLOUD\_LOCATION");

if (\!projectId ||\!location) {  
  throw new Error("GOOGLE\_CLOUD\_PROJECT and GOOGLE\_CLOUD\_LOCATION must be set for Vertex AI.");  
}

// Initialize the client for the Vertex AI platform  
const ai \= new GoogleGenAI({  
  vertexai: true,  
  project: projectId,  
  location: location,  
});

console.log(\`Client initialized for Vertex AI in project '${projectId}' at '${location}'.\`);

### **3.2 A Tour of the Submodules (ai.\*)**

The GoogleGenAI client instance (ai) provides access to all API features through its submodules. This modular design organizes the SDK's capabilities into clear, distinct categories.

* ai.models: This is the primary workhorse for all content generation. It houses the methods for interacting directly with the generative models, such as generateContent for text and multimodal prompts, generateImages for image creation, and embedContent for creating vector embeddings. It is also used to retrieve metadata about available models.  
* ai.chats: This submodule is dedicated to building stateful, multi-turn conversational applications. It allows you to create local Chat objects that automatically manage conversation history, simplifying the process of building chatbots and agents that can remember previous interactions.  
* ai.files: Essential for multimodal applications, this submodule provides tools for managing files. You can upload files (images, documents, audio, video) to the API using ai.files.upload. The API returns a URI for the uploaded file, which can then be referenced in your prompts. This is highly efficient for large files or files that are used repeatedly, as it minimizes bandwidth usage.  
* ai.caches: This submodule offers a powerful mechanism for performance and cost optimization. It allows you to create and manage Cache objects for large prompt prefixes. When you frequently use the same large piece of context (e.g., a long document in a RAG system), caching it can significantly reduce latency and token consumption on subsequent API calls.  
* ai.live: This advanced submodule enables real-time, interactive sessions with Gemini models. It supports streaming text, audio, and video input, with corresponding text or audio output. This is ideal for dynamic applications that require immediate feedback or continuous, low-latency interaction, such as live translation or interactive assistants.

## **Chapter 4: Mastering Content Generation with ai.models**

The ai.models submodule is the core interface for all generative tasks. This chapter provides detailed, Deno-specific examples for text generation, streaming, multimodal inputs, and advanced configuration.

### **4.1 Text Generation and Streaming**

#### **Simple Text Generation**

The most fundamental use of the SDK is generating text from a text prompt. The ai.models.generateContent method is the primary tool for this task. It takes a request object specifying the model and the content to be processed.

The following Deno script demonstrates a basic text generation call:

TypeScript

// text\_generation.ts  
import { GoogleGenAI } from "npm:@google/genai";

const apiKey \= Deno.env.get("GEMINI\_API\_KEY");  
if (\!apiKey) throw new Error("API Key not found");

const ai \= new GoogleGenAI({ apiKey });

async function generateText() {  
  const model \= "gemini-2.0-flash";  
  const prompt \= "Explain the concept of Rayleigh scattering and why it makes the sky appear blue.";

  console.log("Generating content...");  
  const response \= await ai.models.generateContent({  
    model: model,  
    contents: prompt,  
  });

  console.log(response.text);  
}

generateText().catch(console.error);

To run this file:  
deno run \--allow-env \--allow-net text\_generation.ts

#### **Streaming Responses in Deno**

For applications that require real-time feedback, such as chatbots or code completion tools, waiting for the entire response to be generated can lead to a poor user experience. The ai.models.generateContentStream method solves this by returning an AsyncIterable that yields chunks of the response as they are generated by the model.

Deno's first-class support for modern JavaScript features makes handling these streams particularly elegant using the for await...of loop.

TypeScript

// streaming\_generation.ts  
import { GoogleGenAI } from "npm:@google/genai";

const apiKey \= Deno.env.get("GEMINI\_API\_KEY");  
if (\!apiKey) throw new Error("API Key not found");

const ai \= new GoogleGenAI({ apiKey });

async function streamText() {  
  const model \= "gemini-2.0-flash";  
  const prompt \= "Write a long, detailed poem about the journey of a single water molecule.";

  console.log("Streaming content...\\n");  
  const responseStream \= await ai.models.generateContentStream({  
    model: model,  
    contents: prompt,  
  });

  // The for await...of loop is the idiomatic way to handle async iterables in Deno/JS.  
  for await (const chunk of responseStream) {  
    // The Deno.stdout.write is used for continuous output without newlines.  
    const chunkText \= chunk.text;  
    const encoder \= new TextEncoder();  
    await Deno.stdout.write(encoder.encode(chunkText));  
  }  
  console.log("\\n\\n--- Stream Complete \---");  
}

streamText().catch(console.error);

To run this file:  
deno run \--allow-env \--allow-net streaming\_generation.ts

### **4.2 Multimodal Prompts: Text and Images**

Gemini models are inherently multimodal, capable of processing and reasoning about various data types simultaneously. The SDK provides two primary methods for including images in your prompts.

#### **Method 1: Inline Image Data (Base64)**

For small, single-use images, you can embed them directly into the request as a Base64-encoded string. This is convenient as it doesn't require a separate upload step. The following Deno example demonstrates how to read a local image file, encode it, and send it along with a text prompt.

TypeScript

// multimodal\_inline.ts  
import { GoogleGenAI } from "npm:@google/genai";  
import { toBase64 } from "https://deno.land/std@0.224.0/encoding/base64.ts";

const apiKey \= Deno.env.get("GEMINI\_API\_KEY");  
if (\!apiKey) throw new Error("API Key not found");

const ai \= new GoogleGenAI({ apiKey });

// Helper function to read a file and convert it to a Base64 string  
async function fileToGenerativePart(path: string, mimeType: string) {  
  const imageBytes \= await Deno.readFile(path);  
  return {  
    inlineData: {  
      data: toBase64(imageBytes),  
      mimeType,  
    },  
  };  
}

async function runMultimodal() {  
  // Use a model that supports image input, like gemini-2.5-pro  
  const model \= "gemini-2.5-pro";

  const prompt \= "Describe what is happening in this image. What is the object and what might it be used for?";  
  const imagePart \= await fileToGenerativePart("jetpack.png", "image/png");

  // To run this, first download the image:  
  // curl \-o jetpack.png https://storage.googleapis.com/generativeai-downloads/data/jetpack.png

  console.log("Sending multimodal prompt with inline image...");  
  const response \= await ai.models.generateContent({  
    model: model,  
    // The 'contents' array can mix text and image parts  
    contents: \[prompt, imagePart\],  
  });

  console.log(response.text);  
}

runMultimodal().catch(console.error);

To run this file, you first need the image and then execute the script with file read permissions:  
curl \-o jetpack.png https://storage.googleapis.com/generativeai-downloads/data/jetpack.png  
deno run \--allow-env \--allow-net \--allow-read=jetpack.png multimodal\_inline.ts

#### **Method 2 (Recommended): Using ai.files API**

For larger files, or files you intend to reuse across multiple prompts, the ai.files API is the superior method. It involves a two-step process: first, upload the file to the API; second, reference the file's URI in your generation request. This approach is more efficient as the file data is only transferred once.

TypeScript

// multimodal\_files\_api.ts  
import { GoogleGenAI } from "npm:@google/genai";

const apiKey \= Deno.env.get("GEMINI\_API\_KEY");  
if (\!apiKey) throw new Error("API Key not found");

const ai \= new GoogleGenAI({ apiKey });

async function runMultimodalWithFilesAPI() {  
  const model \= "gemini-2.5-pro";  
  const imagePath \= "jetpack.png"; // Ensure this file exists

  console.log(\`Uploading file: ${imagePath}...\`);  
  // Step 1: Upload the file using the ai.files submodule  
  const uploadedFile \= await ai.files.upload({  
    file: imagePath,  
    config: { mimeType: "image/png" },  
  });  
  console.log(\`File uploaded successfully. URI: ${uploadedFile.uri}\`);

  const prompt \= "Based on the provided file, write a creative product description for an e-commerce website.";

  // Step 2: Reference the uploaded file in the generateContent call  
  const response \= await ai.models.generateContent({  
    model: model,  
    contents:,  
      },  
    \],  
  });

  console.log("\\n--- Product Description \---");  
  console.log(response.text);  
}

runMultimodalWithFilesAPI().catch(console.error);

To run this file:  
deno run \--allow-env \--allow-net \--allow-read=jetpack.png multimodal\_files\_api.ts

### **4.3 Advanced Generation Controls**

The SDK provides fine-grained control over the model's generation process through the config object in the request.

#### **Model Parameters (generationConfig)**

These parameters allow you to influence the creativity, length, and structure of the model's output.

* temperature: A value between 0.0 and 2.0 that controls randomness. Lower values (e.g., 0.2) make the output more deterministic and focused, while higher values (e.g., 1.0) encourage more creative and diverse responses.  
* topK: An integer that restricts the model's selection to the K most likely next tokens. A topK of 1 is greedy decoding.  
* topP: A value between 0.0 and 1.0 that uses nucleus sampling. The model considers tokens from most to least probable until their cumulative probability reaches the topP value.  
* maxOutputTokens: An integer to set the maximum length of the generated response.  
* stopSequences: An array of strings that, if generated, will cause the model to stop its output.

TypeScript

// generation\_config.ts  
import { GoogleGenAI } from "npm:@google/genai";

//... (API key setup)...  
const ai \= new GoogleGenAI({ apiKey: Deno.env.get("GEMINI\_API\_KEY")\! });

async function configuredGeneration() {  
  const response \= await ai.models.generateContent({  
    model: "gemini-2.0-flash",  
    contents: "Write a tagline for a new brand of coffee.",  
    config: {  
      temperature: 0.9,  
      topK: 1,  
      topP: 1,  
      maxOutputTokens: 15,  
      stopSequences: \["."\],  
    },  
  });  
  console.log(response.text);  
}

configuredGeneration();

#### **Safety Settings (safetySettings)**

The Gemini API includes built-in safety filters to block harmful content. You can adjust the sensitivity of these filters for specific harm categories using the safetySettings property.2

* category: The type of harm to configure (e.g., HARM\_CATEGORY\_HARASSMENT, HARM\_CATEGORY\_HATE\_SPEECH).  
* threshold: The blocking level (e.g., BLOCK\_LOW\_AND\_ABOVE, BLOCK\_MEDIUM\_AND\_ABOVE, BLOCK\_ONLY\_HIGH, BLOCK\_NONE).

TypeScript

// safety\_settings.ts  
import { GoogleGenAI, HarmCategory, HarmBlockThreshold } from "npm:@google/genai";

//... (API key setup)...  
const ai \= new GoogleGenAI({ apiKey: Deno.env.get("GEMINI\_API\_KEY")\! });

async function safeGeneration() {  
  const prompt \= "This is a potentially sensitive prompt.";  
  const response \= await ai.models.generateContent({  
    model: "gemini-2.0-flash",  
    contents: prompt,  
    config: {  
      safetySettings:,  
    },  
  });

  // Check if the response was blocked  
  if (response.candidates.finishReason \=== "SAFETY") {  
    console.log("The response was blocked due to safety settings.");  
    console.log(response.candidates.safetyRatings);  
  } else {  
    console.log(response.text);  
  }  
}

safeGeneration();

## **Chapter 5: Building Conversational AI with ai.chats**

While ai.models.generateContent is suitable for single-turn requests, building interactive chatbots requires managing conversation history. The ai.chats submodule is specifically designed for this purpose, providing a stateful interface that simplifies multi-turn interactions.

### **5.1 Creating and Managing Stateful Conversations**

The ai.chats.create method instantiates a Chat object. This object automatically maintains the history of the conversation, appending both user messages and model responses. This means you don't have to manually re-send the entire conversation history with each new message.

You can start a new chat from scratch or initialize it with a pre-existing history, which is useful for resuming conversations.

TypeScript

// basic\_chat.ts  
import { GoogleGenAI } from "npm:@google/genai";

const apiKey \= Deno.env.get("GEMINI\_API\_KEY");  
if (\!apiKey) throw new Error("API Key not found");

const ai \= new GoogleGenAI({ apiKey });

async function runChat() {  
  const chat \= ai.chats.create({  
    model: "gemini-2.0-flash",  
    history: \[  
      {  
        role: "user",  
        parts: \[{ text: "Hello, I have some questions about astrophysics." }\],  
      },  
      {  
        role: "model",  
        parts: \[{ text: "Of course\! I'd be happy to help. What's your first question?" }\],  
      },  
    \],  
  });

  console.log("--- Starting Chat \---");

  const msg1 \= "What is a neutron star?";  
  console.log(\`\> User: ${msg1}\`);  
  const response1 \= await chat.sendMessage({ message: msg1 });  
  console.log(\`\> Model: ${response1.text}\`);

  const msg2 \= "And how is it different from a black hole?";  
  console.log(\`\> User: ${msg2}\`);  
  const response2 \= await chat.sendMessage({ message: msg2 });  
  console.log(\`\> Model: ${response2.text}\`);

  // You can inspect the full history at any time  
  // console.log(JSON.stringify(await chat.getHistory(), null, 2));  
}

runChat().catch(console.error);

To run this file:  
deno run \--allow-env \--allow-net basic\_chat.ts

### **5.2 Streaming Chat Responses**

Just as with single-turn generation, you can stream responses in a chat session to create a more interactive and responsive user experience. The chat.sendMessageStream method returns an async iterable that yields response chunks as they are generated.

This is ideal for creating a "typing" effect in a web UI or for providing immediate feedback in a command-line application.

TypeScript

// streaming\_chat.ts  
import { GoogleGenAI } from "npm:@google/genai";

const apiKey \= Deno.env.get("GEMINI\_API\_KEY");  
if (\!apiKey) throw new Error("API Key not found");

const ai \= new GoogleGenAI({ apiKey });

async function runStreamingChat() {  
  const chat \= ai.chats.create({  
    model: "gemini-2.0-flash",  
  });

  const msg \= "Write a short story about a robot who learns to paint.";  
  console.log(\`\> User: ${msg}\\n\> Model: \`);

  const responseStream \= await chat.sendMessageStream({ message: msg });

  const encoder \= new TextEncoder();  
  for await (const chunk of responseStream) {  
    await Deno.stdout.write(encoder.encode(chunk.text));  
  }  
  console.log("\\n--- Chat Complete \---");  
}

runStreamingChat().catch(console.error);

To run this file:  
deno run \--allow-env \--allow-net streaming\_chat.ts

### **5.3 A Complete Deno Chatbot Example**

To demonstrate a practical application, here is the full source code for a simple, interactive command-line chatbot built with Deno. This example sets up a continuous loop that reads user input from the terminal, sends it to the chat session, streams the model's response back, and maintains the conversation history until the user types "exit".

TypeScript

// deno\_chatbot.ts  
import { GoogleGenAI } from "npm:@google/genai";  
import { readLines } from "https://deno.land/std@0.224.0/io/read\_lines.ts";

const apiKey \= Deno.env.get("GEMINI\_API\_KEY");  
if (\!apiKey) throw new Error("API Key not found");

const ai \= new GoogleGenAI({ apiKey });  
const encoder \= new TextEncoder();  
const decoder \= new TextDecoder();

async function main() {  
  const chat \= ai.chats.create({  
    model: "gemini-2.0-flash",  
    // Optional: Add a system instruction to guide the bot's personality  
    config: {  
      systemInstruction: "You are a friendly and helpful assistant named Deni. You answer questions concisely and love the Deno runtime.",  
    },  
  });

  console.log("Chat with Deni\! Type 'exit' to end the conversation.");

  while (true) {  
    await Deno.stdout.write(encoder.encode("\> User: "));  
    const { value: userInput } \= await readLines(Deno.stdin).next();

    if (userInput \=== null |  
| userInput.toLowerCase() \=== "exit") {  
      console.log("\\nDeni: Goodbye\!");  
      break;  
    }

    try {  
      const responseStream \= await chat.sendMessageStream({ message: userInput });  
      await Deno.stdout.write(encoder.encode("\> Deni: "));  
      for await (const chunk of responseStream) {  
        await Deno.stdout.write(encoder.encode(chunk.text));  
      }  
      await Deno.stdout.write(encoder.encode("\\n"));  
    } catch (error) {  
      console.error("\\nAn error occurred:", error.message);  
    }  
  }  
}

main();

To run this interactive chatbot:  
deno run \--allow-env \--allow-net \--unstable-stdin deno\_chatbot.ts

## **Chapter 6: Unlocking Advanced Capabilities**

Beyond text and chat, the @google/genai SDK offers a suite of advanced features that enable more complex and powerful applications. This chapter covers function calling for building agents, semantic embeddings for information retrieval, context caching for optimization, and an introduction to the real-time Live API.

### **6.1 Function Calling: Giving Your AI Tools**

Function calling transforms the language model from a simple text generator into a reasoning engine that can use external tools. Instead of just responding with text, the model can ask to execute a function you've defined in your Deno code, receive the result, and then formulate a final, tool-informed answer. This is the foundation for building agents that can interact with APIs, databases, or any other external system.

The workflow involves several key steps 3:

1. **Define the Tool (Function Declaration):** You describe your Deno function to the model using a specific JSON schema, known as a FunctionDeclaration. This schema includes the function's name, a description of what it does, and the expected parameters.  
2. **Make the Initial Call:** You send the user's prompt to the model along with the FunctionDeclaration in the tools part of the request configuration.  
3. **Handle the Response:** The SDK checks the model's response. If the model decides to use the tool, the response will contain a functionCall object with the name of the function to execute and the arguments it has inferred from the user's prompt.  
4. **Execute Locally:** Your Deno application calls the actual function with the provided arguments.  
5. **Return the Result:** You make a second call to the model, sending back the original history plus a new functionResponse part containing the output from your local function. The model then uses this data to generate a final, natural language response for the user.

Here is a complete Deno example demonstrating this flow:

TypeScript

// function\_calling.ts  
import { GoogleGenAI, Type } from "npm:@google/genai";

const apiKey \= Deno.env.get("GEMINI\_API\_KEY");  
if (\!apiKey) throw new Error("API Key not found");

const ai \= new GoogleGenAI({ apiKey });

// Step 1: Define the function and its declaration  
const getCurrentWeather \= (location: string) \=\> {  
  console.log(\` Fetching weather for ${location}\`);  
  if (location.toLowerCase().includes("tokyo")) {  
    return { weather: "sunny", temperature: "22°C" };  
  } else if (location.toLowerCase().includes("london")) {  
    return { weather: "cloudy", temperature: "15°C" };  
  }  
  return { weather: "unknown", temperature: "N/A" };  
};

const weatherTool: any \= {  
  functionDeclarations:,  
      },  
    },  
  \],  
};

async function runFunctionCalling() {  
  const model \= "gemini-2.0-flash";  
  const prompt \= "What's the weather like in London?";  
  const contents \= \[{ role: "user", parts: \[{ text: prompt }\] }\];

  // Step 2: Make the initial call with the tool definition  
  console.log("User asks:", prompt);  
  const initialResponse \= await ai.models.generateContent({  
    model: model,  
    contents: contents,  
    config: { tools: },  
  });

  // Step 3: Handle the function call response  
  const functionCall \= initialResponse.functionCalls?.;  
  if (functionCall) {  
    console.log("Model wants to call a function:", functionCall);  
    const { name, args } \= functionCall;

    if (name \=== "getCurrentWeather") {  
      // Step 4: Execute the local Deno function  
      const toolResult \= getCurrentWeather(args.location);

      // Step 5: Return the result to the model  
      const finalResponse \= await ai.models.generateContent({  
        model: model,  
        contents: \[  
         ...contents,  
          { role: "model", parts: \[{ functionCall }\] }, // Previous model turn  
          {  
            role: "user", // Role is 'user' for function responses  
            parts:,  
          },  
        \],  
        config: { tools: },  
      });

      console.log("Final model response:", finalResponse.text);  
    }  
  } else {  
    console.log("Model responded directly:", initialResponse.text);  
  }  
}

runFunctionCalling().catch(console.error);

To run this file:  
deno run \--allow-env \--allow-net function\_calling.ts

### **6.2 Semantic Power with Embeddings**

Embeddings are numerical vector representations of text that capture its semantic meaning. Text with similar meanings will have vectors that are "closer" together in the vector space. This is the technology that powers semantic search, retrieval-augmented generation (RAG), classification, and clustering.

The ai.models.embedContent method is used to generate these embeddings.

TypeScript

// embeddings.ts  
import { GoogleGenAI } from "npm:@google/genai";

const apiKey \= Deno.env.get("GEMINI\_API\_KEY");  
if (\!apiKey) throw new Error("API Key not found");

const ai \= new GoogleGenAI({ apiKey });

async function generateEmbeddings() {  
  const model \= "text-embedding-004"; // A model specialized for embeddings  
  const texts \=;

  console.log("Generating embeddings for 3 sentences...");  
  const response \= await ai.models.embedContent({  
    model: model,  
    contents: texts,  
  });

  // The response contains an array of embedding objects  
  response.embeddings.forEach((embedding, index) \=\> {  
    console.log(\`\\nEmbedding for: "${texts\[index\]}"\`);  
    console.log(\`Vector dimensions: ${embedding.value.length}\`);  
    console.log(\`First 5 values: \[${embedding.value.slice(0, 5).join(", ")}\]\`);  
  });  
}

generateEmbeddings().catch(console.error);

#### **The Critical task\_type Parameter**

For high-quality results, especially in RAG systems, it is crucial to specify the task\_type in your embedding request. This optimizes the generated embeddings for a specific use case.4

* RETRIEVAL\_QUERY: Use for the user's question.  
* RETRIEVAL\_DOCUMENT: Use for the documents you are searching through.  
* SEMANTIC\_SIMILARITY: For general similarity tasks.  
* CLASSIFICATION: For classification tasks.  
* CLUSTERING: For clustering tasks.

Using different task types for the query and the documents can significantly improve the relevance of search results.

TypeScript

// Example of using task\_type for RAG  
const query \= "How does Deno handle security?";  
const document \= "Deno is a secure runtime for JavaScript and TypeScript. It sandboxes code and requires explicit permissions for file, network, and environment access.";

const queryEmbedding \= await ai.models.embedContent({  
  model: "text-embedding-004",  
  contents: query,  
  config: { taskType: "RETRIEVAL\_QUERY" },  
});

const documentEmbedding \= await ai.models.embedContent({  
  model: "text-embedding-004",  
  contents: document,  
  config: { taskType: "RETRIEVAL\_DOCUMENT" },  
});

### **6.3 Performance and Cost Optimization with ai.caches**

When an application repeatedly uses the same large prompt prefix—such as a long PDF document provided as context for a series of questions—the ai.caches submodule can provide significant performance and cost benefits. It allows you to create a Cache object that stores the processed representation of the initial context, avoiding the need to re-process it on every subsequent call.

TypeScript

// context\_caching.ts  
import { GoogleGenAI } from "npm:@google/genai";

const apiKey \= Deno.env.get("GEMINI\_API\_KEY");  
if (\!apiKey) throw new Error("API Key not found");

const ai \= new GoogleGenAI({ apiKey });

async function runWithCache() {  
  const modelName \= "gemini-1.5-flash";  
  const largeDocumentText \= await Deno.readTextFile("large\_document.txt");

  // Step 1: Create the cache from the large context  
  console.log("Creating context cache...");  
  const cache \= await ai.caches.create({  
    model: modelName,  
    config: {  
      contents: }\],  
      systemInstruction: "You are an expert at analyzing technical documents.",  
    },  
  });  
  console.log(\`Cache created: ${cache.name}\`);

  // Step 2: Use the cache in a subsequent generation call  
  const question \= "Summarize the key findings in this document.";  
  console.log(\`\\nAsking: "${question}" using the cache...\`);  
  const response \= await ai.models.generateContent({  
    model: modelName,  
    contents: question,  
    config: { cachedContent: cache.name },  
  });

  console.log("\\n--- Summary \---");  
  console.log(response.text);  
}

// Create a dummy file for the example  
await Deno.writeTextFile("large\_document.txt", "This is a very long document about the history of computing... (imagine many pages of text here)");  
runWithCache().catch(console.error);

To run this file:  
deno run \--allow-env \--allow-net \--allow-read \--allow-write context\_caching.ts

### **6.4 An Introduction to the Live API (ai.live)**

The ai.live submodule is a preview feature designed for building highly interactive, real-time applications. It establishes a persistent, low-latency connection to the model, supporting streaming of text, audio, and video input, and receiving text or audio output.

While a full implementation is beyond the scope of this guide, the core concepts involve:

1. **Connecting:** Use ai.live.connect() to establish a session, providing a model name and a set of callbacks.  
2. **Callbacks:** Define functions for onopen, onmessage, onerror, and onclose to handle the lifecycle of the connection.  
3. **Sending Content:** Use the session.sendClientContent() method to stream data to the model.  
4. **Handling Turns:** Process messages from the server to manage the back-and-forth interaction, which is often turn-based.  
5. **Closing:** Use session.close() to terminate the session.

This API is ideal for applications like real-time voice assistants, live translation services, or interactive video analysis tools.

## **Chapter 7: API Reference for the Deno Developer**

This chapter provides a concise reference for the most commonly used classes, methods, and types in the @google/genai SDK, tailored for quick consultation during Deno development. This is not an exhaustive list but covers the primary API surface.

### **GoogleGenAI Class**

The main client class that serves as the entry point to the SDK.

Constructor  
new GoogleGenAI(config: GoogleGenAIConfig)

* **config**: An object with the following properties:  
  * apiKey?: string: Your API key from Google AI Studio. Required for the Gemini Developer API.  
  * vertexai?: boolean: Set to true to use the Vertex AI backend.  
  * project?: string: Your Google Cloud Project ID. Required for Vertex AI.  
  * location?: string: Your Google Cloud project location. Required for Vertex AI.  
  * apiVersion?: string: Optionally specify the API version, e.g., 'v1' for stable or 'v1beta' for preview features.

### **ai.models Methods**

The submodule for direct interaction with generative models.

* generateContent(request: GenerateContentRequest): Promise\<GenerateContentResponse\>  
  * Generates content based on a single, non-streaming request.  
  * **request**: An object containing model, contents, and an optional config object for generationConfig and safetySettings.  
* generateContentStream(request: GenerateContentRequest): AsyncIterable\<GenerateContentResponse\>  
  * Generates content and streams the response back in chunks.  
  * Returns an async iterable suitable for a for await...of loop.  
* embedContent(request: EmbedContentRequest): Promise\<EmbedContentResponse\>  
  * Generates vector embeddings for a given piece of text or list of texts.  
  * **request**: An object containing model, contents, and an optional config object for taskType and outputDimensionality.  
* countTokens(request: CountTokensRequest): Promise\<CountTokensResponse\>  
  * Calculates the number of tokens in a given prompt without making a full generation call.

### **ai.chats Methods**

The submodule for creating and managing stateful conversations.

* create(request: CreateChatRequest): Chat  
  * Creates a new Chat instance.  
  * **request**: An object containing model and an optional history array to start the chat with pre-existing context.  
* chat.sendMessage(request: SendMessageRequest): Promise\<GenerateContentResponse\>  
  * Sends a message in an existing chat session and returns the full response once generated.  
* chat.sendMessageStream(request: SendMessageRequest): AsyncIterable\<GenerateContentResponse\>  
  * Sends a message and streams the response back in chunks.

### **ai.files Methods**

The submodule for managing files for multimodal prompts.

* upload(request: UploadFileRequest): Promise\<File\>  
  * Uploads a file to the API.  
  * **request**: An object containing the file path and an optional config object with mimeType and displayName.  
  * Returns a File object containing the uri and mimeType of the uploaded file.

### **Key Interfaces and Types**

A brief look at the data structures you will frequently use.

* **Content**: Represents a single turn in a conversation. It has a role ('user' or 'model') and an array of parts.  
* **Part**: A piece of content within a Content object. Can be one of the following:  
  * { text: string }  
  * { inlineData: { data: string; mimeType: string } } (for Base64 data)  
  * { fileData: { fileUri: string; mimeType: string } } (for uploaded files)  
  * { functionCall: FunctionCall }  
  * { functionResponse: FunctionResponse }  
* **FunctionDeclaration**: The JSON schema used to describe a tool to the model.  
* **GenerationConfig**: An object containing parameters like temperature, topK, topP, and maxOutputTokens.  
* **SafetySetting**: An object specifying a category and a threshold for content filtering.

## **Conclusion: Building the Future on Deno with Gemini**

The @google/genai SDK marks a significant advancement in the developer toolkit for generative AI, offering a unified, powerful, and forward-looking interface to Google's most advanced models. This guide has demonstrated that the Deno runtime, with its modern features and robust npm compatibility, is not just a viable platform but an exceptionally well-suited environment for harnessing the full potential of this new SDK.

The architectural synergy between the SDK's explicit, modular design and Deno's philosophy of security and clarity creates a development experience that is both productive and intuitive. Developers can confidently build applications that start as simple prototypes using the Gemini Developer API and scale seamlessly to enterprise-grade solutions on Vertex AI, all from a consistent and Deno-native codebase.

From basic text generation and streaming to advanced applications involving multimodal reasoning, stateful chat, and agentic function calling, the combination of @google/genai and Deno provides a formidable foundation. As the capabilities of models like Gemini continue to expand, this SDK will be the gateway to those innovations. By embracing this modern toolset, Deno developers are well-equipped to build the next generation of intelligent, responsive, and sophisticated AI-powered applications.

For continued learning and to stay updated, developers are encouraged to consult the following official resources:

* **Official SDK Documentation:** [googleapis.github.io/js-genai/](https://googleapis.github.io/js-genai/)  
* **GitHub Repository:** [github.com/googleapis/js-genai](https://github.com/googleapis/js-genai)  
* **Google AI for Developers Community:** [discuss.ai.google.dev](https://discuss.ai.google.dev)

#### **Works cited**

1. Upgrade to the Google Gen AI SDK | Gemini API | Google AI for ..., accessed June 18, 2025, [https://ai.google.dev/gemini-api/docs/migrate](https://ai.google.dev/gemini-api/docs/migrate)  
2. Gemini API libraries | Google AI for Developers, accessed June 18, 2025, [https://ai.google.dev/gemini-api/docs/libraries](https://ai.google.dev/gemini-api/docs/libraries)  
3. Function Calling with the Gemini API | Google AI for Developers, accessed June 18, 2025, [https://ai.google.dev/gemini-api/docs/function-calling](https://ai.google.dev/gemini-api/docs/function-calling)  
4. Embeddings | Gemini API | Google AI for Developers, accessed June 18, 2025, [https://ai.google.dev/gemini-api/docs/embeddings](https://ai.google.dev/gemini-api/docs/embeddings)