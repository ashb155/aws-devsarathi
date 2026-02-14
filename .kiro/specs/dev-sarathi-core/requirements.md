# Requirements Document

## Introduction

Dev-Sarathi is an Indic-language compatible AI co-pilot designed to democratize and simplify the entire software lifecycle—Build, Ship, and Deploy—for India's developers. By dismantling linguistic barriers, it allows users to interact, design logic and fix issues in their native Indian tongues and confidently build and deploy their ideas. It encompasses the transition from Vernacular Learning (Gyan-Setu) to Voice-based Code Generation (Vani-Srijan), Diagnostic Debugging and Analysis (Dosh-Drishti), and Operational Safety (Karma-Kavach).

## Glossary

- **Dev-Sarathi**: The laptop-first vernacular AI extension (VS Code/CLI)
- **Vani-Srijan**: The Pulse-Batch voice interface and code generation engine
- **Bedrock-Orchestrator**: The Claude 3.5 Sonnet-powered reasoning engine through Amazon Bedrock
- **Gyan-Setu-RAG**: The S3 + Bedrock Knowledge Base retrieval system
- **Dosh-Drishti**: The Diagnostic Agent responsible for assisting developers in debugging, analyzing errors and explaining it to them in vernacular languages
- **Karma-Kavach**: The Bedrock Guardrails safety component to catch potentially risky executions and automate docs generation, version control and testing
- **AWS-SSM**: The AWS Systems Manager executor component
- **Pulse-Batch**: A custom latency-optimization technique for streaming Indic voice data, especially helpful for those languages without any streaming support
- **Vernacular_Content**: Indian language text representing realistic developer communication
- **User**: A developer interacting with Dev-Sarathi
- **System**: The Dev-Sarathi core system

## Requirements

### Requirement 1: Laptop-First Interface

**User Story:** As a developer, I want to install Dev-Sarathi as a local extension, so that I can interact with AWS directly from my VS Code environment using my voice.

#### Acceptance Criteria

1. THE System SHALL accept input via a local terminal interface or sidebar input panel
3. THE System SHALL support voice input via microphone for major Indian languages.
4. WHEN audio input is received, THE System SHALL stream audio to Amazon Transcribe (along with AWS Lambda) using Pulse-Batch mechanism in 200ms chunks to simulate near-realtime streaming.
5. THE System SHALL authenticate with AWS using IAM Roles
6. THE System SHALL reject authentication attempts using hardcoded access keys

### Requirement 2: Vernacular Learning (Gyan-Setu)

**User Story:** As a beginner developer, I want to ask conceptual questions in an Indian language, so that I can understand complex cloud services using familiar analogies.

#### Acceptance Criteria

1. WHEN the User asks a question in vernacular Indian language, THE System SHALL retrieve context from the Gyan-Setu-RAG S3 Knowledge Base
2. WHEN technical documentation is retrieved, THE System SHALL synthesize it into a cultural analogy relevant to Indian context
3. THE System SHALL generate the response in the user's preferred vernacular language
4. THE System SHALL include citations or links to the official English documentation in the response

### Requirement 3: Voice-to-Code Generation (Vani-Srijan)

**User Story:** As a developer, I want to describe programming logic or system tasks in my native Indian language so that the system generates and executes the correct code in my preferred tech stack for me in a secure environment.

#### Acceptance Criteria

1. WHEN the User provides a voice command in a vernacular language, THE System SHALL identify the appropriate target programming language  and generate valid, executable code logic.

2. WHEN code is generated, THE System SHALL execute it within an isolated, secure sandbox environment (e.g., ephemeral Docker container or secure runtime) to prevent host system compromise.

3. THE System SHALL complete the full voice-to-result processing loop (Speech-to-Text → Code Generation → Execution) within 3000ms for standard logic queries.

4.  WHEN code execution succeeds, THE System SHALL summarize the output and return a confirmation message in the user's vernacular language.

5.  WHEN code generation or execution fails or is blocked by security policies, THE System SHALL return a descriptive error message in the user's vernacular language explaining why the action failed.

### Requirement 4: Diagnostic Vision (Dosh-Drishti)

**User Story:** As a developer, I want the system to explain technical errors in my code, so that I can fix issues without needing to decipher complex English logs.

#### Acceptance Criteria

1. WHEN an execution error occurs with a non-zero exit code, THE System SHALL intercept the error
2. WHEN an error is intercepted, THE System SHALL route the raw stack trace to the Dosh-Drishti Bedrock Agent
3. THE Dosh-Drishti SHALL generate a pedagogical explanation of the error in the user's vernacular language
4. THE Dosh-Drishti SHALL suggest a specific fix or command to resolve the error
5. THE System SHALL present the vernacular explanation and suggested fix to the user

### Requirement 5: Operational Safety (Karma-Kavach)

**User Story:** As a team lead, I want to prevent my junior developers from accidentally deleting production resources, so that critical data remains safe.

#### Acceptance Criteria

1. WHEN a user intent is received, THE System SHALL evaluate it against Amazon Bedrock Guardrails before execution
2. WHEN a destructive intent is detected (Delete, Terminate, Purge operations), THE System SHALL block the execution immediately
3. WHEN a destructive intent is blocked, THE System SHALL display a high-priority warning in the user's vernacular language
4. WHEN a destructive intent is blocked, THE System SHALL require an explicit override confirmation from the user
5. WHEN an override confirmation is received, THE System SHALL re-evaluate the intent and proceed with execution if confirmed

### Requirement 6: Transcription Accuracy

**User Story:** As a developer speaking in Indic languages, I want my voice commands to be accurately transcribed, so that the system executes the correct actions.

#### Acceptance Criteria

1. WHEN audio is streamed to Amazon Transcribe, THE System SHALL specify the appropriate language model for Indic languages.
2. WHEN the User initiates voice input, THE System SHALL detect the selected language and route the audio stream to the appropriate processing pipeline:

Streaming Pipeline: For Indian languages with native streaming support (e.g. Hindi)
Pulse-Batch Pipeline: For languages with batch-only support (e.g., Tamil ta-IN, Telugu te-IN, Kannada kn-IN, Malayalam ml-IN, Marathi mr-IN, Gujarati gu-IN).

3. WHEN routing to the Pulse-Batch Pipeline, THE System SHALL:

Buffer audio into "micro-batches" based on silence detection (VAD) or a maximum duration of 5 seconds.

Submit these micro-batches to Amazon Transcribe (Batch API) immediately upon buffer closure.

3. WHEN transcription is complete, THE System SHALL validate the transcribed text before processing

4. THE System SHALL achieve a minimum transcription accuracy of 85% for common developer commands in vernacular languages

### Requirement 7: Multi-Language Support

**User Story:** As a developer from any Indian state, I want to use Dev-Sarathi in my preferred language, so that I can work comfortably in my native tongue.

#### Acceptance Criteria

1. THE System SHALL detect the user's preferred language from their input automatically
2. WHEN a user switches languages, THE System SHALL maintain context across language changes


### Requirement 8: Response Formatting

**User Story:** As a developer, I want responses to be clearly formatted and easy to understand, so that I can quickly act on the information provided.

#### Acceptance Criteria

1. WHEN the System generates a response, THE System SHALL format technical terms consistently
2. WHEN the System includes code snippets, THE System SHALL use syntax highlighting appropriate for the command type
3. WHEN the System provides multiple steps, THE System SHALL number them sequentially
4. THE System SHALL separate cultural analogies from technical details visually
5. THE System SHALL display citations as clickable links when rendered in VS Code





### Non-Functional Requirements (NFRs)

1. **Latency & Performance** -

The Pulse-Batch processing overhead for non-streaming languages (e.g., Tamil, Marathi) must not exceed 2 seconds over the standard streaming baseline.

AWS Lambda functions handling the Bedrock-Orchestrator must utilize provisioned concurrency to ensure cold start latency remains under 500ms.

The system must be capable of handling concurrent voice streams from at least 1,000 active sessions without degradation in transcription accuracy.

2. **Security & Compliance** - 

The system must strictly adhere to AWS Least Privilege principles. No long-term access keys shall be stored locally; temporary credentials must be retrieved via AWS SSO or IAM Identity Center.

Generated code execution must occur in a containerized environment with zero network access to the host machine's local file system outside of a specific designated workspace folder.

Voice recordings processed via Vani-Srijan must be ephemeral. Audio data must be deleted immediately after transcription and not stored for model training unless explicitly opted-in by the user.

3. **Reliability & Availability** - 

In the event of Amazon Bedrock service throttling, the system must implement an exponential backoff strategy and notify the user in their vernacular language rather than crashing.

Uptime: The core backend services (Authentication, Knowledge Base Retrieval) must maintain 99.9% availability during business hours (IST).

4. **Usability & Accessibility** - 

The Speech-to-Text engine must be calibrated to handle heavy Indian English accents and mixed-code speech (e.g., "Hinglish" or "Tanglish") with a tolerance for grammatical errors in spoken commands.

The VS Code extension sidebar must render the vernacular response within 100ms of receiving the payload from the backend.



## Architecture


### Vani-Srijan
**Voice Capture**: Amazon Transcribe.

**Processing Engine**: AWS Lambda (running Hybrid Pulse-Batching algorithms).

**Logic Mapping**: Amazon Bedrock (Claude 3.5 Sonnet) for logic-to-syntax conversion.

### Gyan-Setu
**Context Retrieval**: Amazon Bedrock Knowledge Bases.

**Storage**: Amazon S3 (storing documentation and codebase chunks).

**Synthesis**: Amazon Bedrock (Claude 3.5 Sonnet).

### Dosh-Drishti
**Orchestration**: Amazon Bedrock Agents (to plan and execute diagnostic steps).

**Analysis**: Amazon Bedrock (Claude 3.5 Sonnet).

### Karma-Kavach
**Intent Interpretation**: Amazon Bedrock (Claude 3.5 Sonnet).

**Safety Enforcement**: Amazon Bedrock Guardrails (policy checks).

**Secure Execution**: AWS Systems Manager (SSM) (for shell and Git command execution).



## Scope 
### In-Scope 
    - VS Code Extension Prototype
    - Indic Voice Support (hybrid pulse batch and streaming mechanism )
    - Voice to Code Generation (Vani Srijan)
    - Error Debugging (Dosh Drishti)
    - RAG Learner Systems (Gyan Setu)
    - Ops Safety and Automation (Karma Kavach)


## Success Metrics
    - Code generation accuracy
    - Minimal end-to-end latency
    - Indian language coverage
    - Effective debugging efficiency
    - Learning comprehension improvement
    - User feedback related to confidence in building and deploying

