# Design Document: Dev-Sarathi Core

## Overview

Dev-Sarathi Core is a vernacular AI co-pilot system that enables Indian developers to code, interact and build with AWS services using their native languages. The system integrates multiple AWS services (Transcribe, Bedrock, S3, SSM, Guardrails) to provide a seamless voice-to-action pipeline with built-in safety mechanisms and pedagogical error explanations.

The architecture follows a modular pipeline design:
1. **Input Layer** : Captures voice/text input and transcribes to text
2. **Reasoning and Generation Layer** (Vani-Srijan): Interprets vernacular intent and generates code snippets or commands
3. **Knowledge Layer** (Gyan-Setu-RAG): Provides contextual learning and documentation
4. **Safety Layer and Automation** (Karma-Kavach): Validates commands before execution, and automates testing, docs and version control.
5. **Execution Layer** (AWS-SSM): Executes validated commands securely.
6. **Diagnostic Layer** (Dosh-Drishti): Analyzes and explains errors in vernacular

## Architecture

### High-Level Architecture

```mermaid
graph TB
    %% User Input
    User[User: Voice/Text Input]

    %% Input Layer
    subgraph Input_Layer["Input Layer"]
        VSCode[VS Code Extension]
        CLI[Terminal CLI]
        PulseBatch[Pulse-Batch Streamer]
        Transcribe[Amazon Transcribe]
    end

    %% Reasoning Layer
    subgraph Reasoning_Layer["Reasoning Layer"]
        Orchestrator[Bedrock-Orchestrator<br/>Claude 3.5 Sonnet]
    end

    %% Knowledge Layer
    subgraph Knowledge_Layer["Knowledge Layer"]
        RAG[Gyan-Setu-RAG<br/>S3 + Bedrock KB]
    end

    %% Safety Layer
    subgraph Safety_Layer["Safety Layer"]
        Guardrails[Karma-Kavach<br/>Bedrock Guardrails]
    end

    %% Execution Layer
    subgraph Execution_Layer["Execution Layer (Hybrid)"]
        Router{Router}
        LocalExec[Local Terminal Executor<br/>VS Code API]
        SSM[Remote Executor<br/>AWS Systems Manager]
        AWSServices[AWS Cloud Resources]
    end

    %% Diagnostic Layer
    subgraph Diagnostic_Layer["Diagnostic Layer"]
        Drishti[Dosh-Drishti<br/>Bedrock Agent]
    end

    %% Input Flow
    User -->|Voice| VSCode
    User -->|Voice| CLI
    User -->|Text| VSCode
    User -->|Text| CLI

    VSCode -->|Audio Stream| PulseBatch
    CLI -->|Audio Stream| PulseBatch
    PulseBatch -->|200ms Chunks| Transcribe
    Transcribe -->|Vernacular Text| Orchestrator

    VSCode -->|Text| Orchestrator
    CLI -->|Text| Orchestrator

    %% Reasoning & Knowledge
    Orchestrator <-->|Query/Context| RAG

    %% Safety & Routing
    Orchestrator -->|Intent| Guardrails
    Guardrails -->|Blocked Intent| Orchestrator
    Guardrails -->|Validated Intent| Router

    Router -->|Local Task (Git/Docs)| LocalExec
    Router -->|Cloud Task (Deploy)| SSM

    %% Execution & Feedback
    SSM -->|Execute| AWSServices
    AWSServices -->|Success| Orchestrator
    AWSServices -->|Error| Drishti

    LocalExec -->|Success| Orchestrator
    LocalExec -->|Error| Drishti

    %% Final Output
    Drishti -->|Vernacular Explanation| User
    Orchestrator -->|Vernacular Response| User
    Orchestrator -->|Generated Code Snippet| User

```

### Component Interaction Flow

**Learning Query Flow (Gyan-Setu)**:
1. User asks question in vernacular → Orchestrator
2. Orchestrator queries Gyan-Setu-RAG for context
3. RAG retrieves relevant documentation from S3 Knowledge Base
4. Orchestrator synthesizes cultural analogy + technical explanation
5. Response returned in user's vernacular language

**Command Generation and Execution Flow (Vani-Srijan + Karma-Kavach)**:
1. User expresses logic and reasoning in Indian language(eg. "Git commit kardo","for loop mein even numbers hatado") → Pulse-Batch → Transcribe → Orchestrator
2. Orchestrator interprets intent and generates code/ command. Code snippet sent back to user
3. If command, sent to Karma-Kavach for safety validation
4. Karma-Kavach router directs execution:
  Local: Executed via VS Code Terminal (e.g., "Git commit", "Run tests", "Add docs").
  Remote: Executed via AWS SSM (e.g., "Launch EC2", "Create Bucket").
4. If safe: Command executed → Result returned in vernacular
5. If unsafe: Warning displayed, override confirmation required

**Error Handling Flow (Dosh-Drishti)**:
1. Code or command execution (local or remote) fails with a non-zero exit code
2. Error intercepted and routed to Dosh-Drishti
3. Dosh-Drishti analyzes stack trace using Bedrock Agent
4. Pedagogical explanation generated in vernacular
5. If applicable, Dosh-Drishti can suggest updated code snippets or safer command alternatives

## Components and Interfaces

### 1. Input Layer Components

#### Pulse-Batch Streamer
**Purpose**: Optimize voice input latency by streaming audio in small chunks

**Interface**:
```typescript
interface PulseBatchStreamer {
  // Start streaming audio from microphone
  startStream(audioSource: AudioSource): StreamHandle
  
  // Stop streaming and finalize
  stopStream(handle: StreamHandle): void
  
  // Configure chunk size (default 200ms)
  setChunkSize(milliseconds: number): void

   // Voice Activity Detection options
  setVADOptions(options: VADOptions): void
  
  // Max micro-batch duration (ms)
  setBufferDuration(milliseconds: number): void

  // Routing based on language
  routeAudio(language?: VernacularLanguage): 'streaming' | 'pulse-batch'
}

interface VADOptions {
  sensitivity: 'low' | 'medium' | 'high'   // How aggressively to detect silence
  minSilenceDurationMs: number             // Min silence to split batch
  maxBatchDurationMs: number               // Force batch split if too long
}

interface AudioSource {
  deviceId: string     // Unique ID of the microphone
  sampleRate: number   // Audio sample rate in Hz
  channels: number     // Number of audio channels
}

interface StreamHandle {
  id: string
  status: 'active' | 'paused' | 'stopped'   // Current stream status
  startTime?: number   // Track when streaming started
  endTime?: number     // Track when streaming finished
  latency?: number     // Calculate voice-to-text processing latency
}
```

**Behavior**:
- Buffers audio in 200ms chunks
- Streams to Amazon Transcribe via WebSocket
- Handles network interruptions gracefully

#### VS Code Extension Interface
**Purpose**: Provide integrated IDE experience

**Interface**:
```typescript
interface DevSarathiExtension {
  // Activate extension panel
  activate(context: ExtensionContext): void
  
  // Handle text input from panel
  handleTextInput(text: string, language: VernacularLanguage): Promise<Response>
  
  // Handle voice input from microphone
  handleVoiceInput(audioStream: AudioStream): Promise<Response>
  
  // Display response in panel
  displayResponse(response: Response): void
}


interface VernacularLanguageInfo {
  code: string         // e.g., 'hi', 'kn', 'ta', 'te'
  name: string         // e.g., 'Hindi', 'Kannada'
  type?: 'official' | 'slang' | 'mixed'
}
type VernacularLanguage = VernacularLanguageInfo  //example indian languages and mixed slang

interface Response {
  content: string
  language: VernacularLanguage
  citations?: Citation[]
  codeSnippets?: CodeSnippet[]
  format?: ResponseFormat
}

interface ResponseFormat {
  numberedSteps?: boolean
  codeSyntaxHighlight?: string   // Any language or format (e.g., 'go', 'rust', 'html', 'yaml')
  separateAnalogy?: boolean
  clickableCitations?: boolean
}

```

#### Terminal CLI Interface
**Purpose**: Provide command-line access for terminal-first developers

**Interface**:
```typescript
interface DevSarathiCLI {
  // Initialize CLI with AWS credentials
  init(config: AWSConfig): void
  
  // Process text command
  processText(command: string): Promise<CLIResponse>
  
  // Process voice command
  processVoice(): Promise<CLIResponse>
  
  // Set preferred language
  setLanguage(lang: VernacularLanguage): void
}

interface CLIResponse {
  success: boolean
  message: string
  output?: string
  error?: ErrorDetails
}
```

### 2. Reasoning Layer Components

#### Bedrock-Orchestrator
**Purpose**: Central reasoning engine that interprets intent and coordinates all operations

**Interface**:
```typescript
interface BedrockOrchestrator {
  // Process user input and determine intent
  processInput(input: UserInput): Promise<Intent>
  
  // Generate code  command from intent
  generateCommand(intent: Intent): Promise<AWSCommand>
  
  // Query knowledge base for learning requests
  queryKnowledgeBase(query: string, language: VernacularLanguage): Promise<KnowledgeResponse>
  
  // Synthesize response in vernacular
  synthesizeResponse(data: any, language: VernacularLanguage): Promise<string>

  // Detect language automatically if not specified
  detectLanguage(input: string): VernacularLanguage

  // Track current session language for multi-language support
  setSessionLanguage(sessionId: string, language: VernacularLanguage): void

  validateIntent(intent: Intent): Promise<SafetyEvaluation>

}

interface UserInput {
  text: string
  language: VernacularLanguage
  context?: ConversationContext
}

interface Intent {
  type: 'learning' | 'code' | 'command'| 'diagnostic'    // Intent type guides Orchestrator on next action
  action?: AWSAction                                     // Optional AWS-specific action for command intents
  parameters?: Record<string, any>                       // Any additional parameters needed for execution
  isDestructive: boolean,                                // True if action could modify/destroy resources
  targetLayer?: 'local' | 'cloud' | 'learning'           // Determines execution context
}

interface AWSCommand {
  service: string                                       // AWS service name (e.g., S3, EC2)
  operation: string                                     // Action to perform (e.g., 'CreateBucket')
  parameters: Record<string, any>                       // Parameters required for the action
  cliString: string                                     // CLI equivalent command string, useful for logging and direct execution
  generatedAt?: Date
  processedAt?: Date
}
```

### 3. Knowledge Layer Components

#### Gyan-Setu-RAG
**Purpose**: Retrieve and provide contextual documentation for learning queries

**Interface**:
```typescript
interface GyanSetuRAG {
  // Query knowledge base with semantic search
  query(question: string, language: VernacularLanguage): Promise<KnowledgeResponse>
  
  // Retrieve specific documentation
  getDocumentation(service: string, topic: string): Promise<Documentation>
  
  // Generate cultural analogy
  generateAnalogy(concept: string, language: VernacularLanguage): Promise<Analogy>
}

interface KnowledgeResponse {
  answer: string                                         // Main answer text
  analogy?: Analogy                                      // Optional cultural analogy to explain concept
  citations: Citation[]                                  // References for documentation and credibility
  relatedTopics: string[]                                // Topics related to the query for further learning
  clickableCitations?: boolean
}

interface Analogy {
  concept: string
  analogy: string
  explanation: string
  language: VernacularLanguage
}

interface Citation {
  title: string
  url: string
  excerpt: string
}
```

### 4. Safety Layer Components

#### Karma-Kavach (Bedrock Guardrails)
**Purpose**: Validate commands for safety before execution

**Interface**:
```typescript
interface KarmaKavach {
  // Evaluate intent for safety
  evaluateIntent(intent: Intent): Promise<SafetyEvaluation>
  
  // Check if command is destructive
  isDestructive(command: AWSCommand): boolean
  
  // Request override confirmation
  requestOverride(command: AWSCommand, language: VernacularLanguage): Promise<boolean>
}

interface SafetyEvaluation {
  safe: boolean                                     // True if intent/command passes safety checks
  reason?: string                                   // Why it was marked unsafe (if applicable)
  severity: 'low' | 'medium' | 'high' | 'critical'  // Severity of potential risk
  requiresOverride: boolean                         // True if user confirmation needed to proceed
  warningMessage?: string                           // Message to show user when override is needed
}
```

**Destructive Operations**:
- Delete operations (DeleteBucket, DeleteObject, TerminateInstances)
- Purge operations (PurgeQueue, EmptyBucket)
- Terminate operations (TerminateInstances, DeleteStack)
- Modify operations on production resources (UpdateStack, ModifyDBInstance)

### 5. Execution Layer Components

#### AWS-SSM Executor
**Purpose**: Execute validated AWS CLI commands securely

**Interface**:
```typescript
interface AWSSSMExecutor {
  // Execute command via SSM
  executeCommand(command: AWSCommand, target: ExecutionTarget): Promise<ExecutionResult>
  
  // Get command execution status
  getExecutionStatus(executionId: string): Promise<ExecutionStatus>
  
  // Cancel running execution
  cancelExecution(executionId: string): Promise<void>
}

interface ExecutionTarget {
  instanceId?: string
  role: string
  region: string
}

interface ExecutionResult {
  success: boolean           // True if execution completed successfully
  output: string             // Standard output from command or script
  exitCode: number           // Return code of execution, 0 = success
  executionId: string        // Unique ID to track execution
  duration: number           // Time taken to complete command in milliseconds
  responseLanguage?: VernacularLanguage
}

interface ExecutionStatus {
  status: 'pending' | 'running' | 'success' | 'failed' | 'cancelled'
  output?: string
  error?: string
}
```

### 6. Diagnostic Layer Components

#### Dosh-Drishti (Diagnostic Agent)
**Purpose**: Analyze errors and provide vernacular explanations

**Interface**:
```typescript
interface DoshDrishti {
  // Analyze error and generate explanation
  analyzeError(error: ExecutionError, language: VernacularLanguage): Promise<ErrorExplanation>
  
  // Suggest fix for error
  suggestFix(error: ExecutionError, language: VernacularLanguage): Promise<Fix>
  
  // Generate pedagogical explanation
  explainConcept(errorType: string, language: VernacularLanguage): Promise<string>
}

interface ExecutionError {
  code: string               // Can be compiler/interpreter code
  message: string
  stackTrace?: string        // Optional for simpler scripts
  command?: AWSCommand       // Optional for cloud tasks
  codeSnippet?: string       // Optional for local code
  exitCode?: number          // Optional for local executions
}

interface ErrorExplanation {
  summary: string
  detailedExplanation: string
  rootCause: string
  language: VernacularLanguage
  clickableCitations?: boolean
}

interface Fix {
  description: string
  suggestedCommand?: string
  steps: string[]
  preventionTips: string[]
}
```

## Data Models

### User Session
```typescript
interface UserSession {
  sessionId: string                                     // Unique session ID for tracking
  userId: string                                        // User identifier
  sessionLanguages?: Record<string, VernacularLanguage>
  preferredLanguage: VernacularLanguage                 // Language preference for responses
  conversationHistory: Message[]                        // Log of messages in session
  awsCredentials: AWSCredentials                        // AWS access info for commands
  createdAt: Date                                       // Timestamp when session started
  lastActivity: Date                                    // Last activity timestamp
  lastIntentLatencyMs?: number
  overrides?: Record<string, boolean>                   // intentId → override confirmed
}

interface Message {
  id: string
  role: 'user' | 'assistant'             // Who sent the message
  content: string
  language: VernacularLanguage
  timestamp: Date
  intent?: Intent                        // Optional, intent detected for this message
  command?: AWSCommand                   // Optional, command generated
  generatedCode?: string                 // Optional, code snippet generated
  diagnosticFeedback?: string            // Optional, error analysis feedback
}
```

### AWS Credentials
```typescript
interface AWSCredentials {
  type: 'iam-role' | 'profile'
  roleArn?: string
  profileName?: string
  region: string
  sessionToken?: string
  expiresAt?: Date
}
```

### Transcription Result
```typescript
interface TranscriptionResult {
  text: string
  confidence: number
  language: string
  alternatives?: TranscriptionAlternative[]
  duration: number
}

interface TranscriptionAlternative {
  text: string
  confidence: number
}
```

### Knowledge Base Entry
```typescript
interface KnowledgeEntry {
  id: string
  service: string
  topic: string
  content: string
  vernacularTranslations: Record<VernacularLanguage, string>
  analogies: Record<VernacularLanguage, Analogy>
  citations: Citation[]
  lastUpdated: Date
}
```

### Execution Log
```typescript
interface ExecutionLog {
  id: string
  sessionId: string
  command?: AWSCommand  
  codeSnippet?: string    
  intent: Intent
  safetyEvaluation?: SafetyEvaluation
  executionResult?: ExecutionResult
  diagnosticFeedback?: string
  timestamp: Date
  userOverride?: boolean
  executedBy?: 'local' | 'cloud' | 'manual'
  responseLanguage?: VernacularLanguage
  automationType?: 'git' | 'docs' | 'test' | 'deploy'
}
```
