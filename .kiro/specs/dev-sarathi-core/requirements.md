# Requirements Document

## Introduction

Dev-Sarathi is an Indic-language compatible AI co-pilot designed to democratize the entire software lifecycle—Build, Ship, and Deploy—for India's developers. By dismantling linguistic barriers, it allows users to interact in their native tongue and confidently build and deploy their ideas. It encompasses the transition from Vernacular Learning (Gyan-Setu) to Voice-based Code Generation (Vani-Srijan), Diagnostic Vision (Dosh-Drishti), and Operational Safety (Karma-Kavach).

## Glossary

- **Dev-Sarathi**: The laptop-first vernacular AI extension (VS Code/CLI)
- **Vani-Srijan**: The Pulse-Batch voice interface and code generation engine
- **Bedrock-Orchestrator**: The Claude 3.5 Sonnet-powered reasoning engine
- **Gyan-Setu-RAG**: The S3 + Bedrock Knowledge Base retrieval system
- **Dosh-Drishti**: The Diagnostic Agent responsible for analyzing errors and explaining them in vernacular languages
- **Karma-Kavach**: The Bedrock Guardrails safety component
- **AWS-SSM**: The AWS Systems Manager executor component
- **Pulse-Batch**: A custom latency-optimization technique for streaming Indic voice data
- **Vernacular_Content**: Indian language text representing realistic developer communication
- **User**: A developer interacting with Dev-Sarathi
- **System**: The Dev-Sarathi core system

## Requirements

### Requirement 1: Laptop-First Interface (Vani-Srijan)

**User Story:** As a developer, I want to install Dev-Sarathi as a local extension, so that I can interact with AWS directly from my VS Code environment using my voice.

#### Acceptance Criteria

1. THE System SHALL accept input via a local terminal interface or VS Code extension panel
2. THE System SHALL support text input in Hinglish and Kannada languages
3. THE System SHALL support voice input via microphone
4. WHEN audio input is received, THE System SHALL stream audio to Amazon Transcribe using Pulse-Batch in 200ms chunks
5. THE System SHALL authenticate with AWS using IAM Roles
6. THE System SHALL reject authentication attempts using hardcoded access keys

### Requirement 2: Vernacular Learning (Gyan-Setu)

**User Story:** As a beginner developer, I want to ask conceptual questions in Hindi, so that I can understand complex cloud services using familiar analogies.

#### Acceptance Criteria

1. WHEN the User asks a question in vernacular language, THE System SHALL retrieve context from the Gyan-Setu-RAG S3 Knowledge Base
2. WHEN technical documentation is retrieved, THE System SHALL synthesize it into a cultural analogy relevant to Indian context
3. THE System SHALL generate the response in the user's preferred vernacular language
4. THE System SHALL include citations or links to the official English documentation in the response

### Requirement 3: Voice-to-Code Execution (Vani-Srijan)

**User Story:** As a developer, I want to speak infrastructure commands in "Hinglish", so that the system generates and executes the correct AWS CLI commands for me.

#### Acceptance Criteria

1. WHEN the User provides a voice command in vernacular language, THE System SHALL map the vernacular intent to valid executable AWS CLI commands
2. WHEN a command is generated, THE System SHALL execute it using AWS-SSM for secure server-side execution
3. THE System SHALL complete voice-to-action processing within 3000ms for standard queries
4. WHEN command execution succeeds, THE System SHALL return a confirmation message in the user's vernacular language
5. WHEN command execution fails, THE System SHALL return an error message in the user's vernacular language

### Requirement 4: Diagnostic Vision (Dosh-Drishti)

**User Story:** As a developer, I want the system to explain technical errors in my language, so that I can fix issues without needing to decipher complex English logs.

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

**User Story:** As a developer speaking in Hinglish, I want my voice commands to be accurately transcribed, so that the system executes the correct actions.

#### Acceptance Criteria

1. WHEN audio is streamed to Amazon Transcribe, THE System SHALL specify the appropriate language model for Indic languages
2. THE System SHALL handle code-mixed speech (Hinglish, Kanglish) appropriately
3. WHEN transcription is complete, THE System SHALL validate the transcribed text before processing
4. THE System SHALL achieve a minimum transcription accuracy of 85% for common developer commands in vernacular languages

### Requirement 7: Multi-Language Support

**User Story:** As a developer from any Indian state, I want to use Dev-Sarathi in my preferred language, so that I can work comfortably in my native tongue.

#### Acceptance Criteria

1. THE System SHALL support Hindi as a primary vernacular language
2. THE System SHALL support Kannada as a primary vernacular language
3. THE System SHALL support Hinglish (Hindi-English code-mixed) as a primary input mode
4. WHEN a user switches languages, THE System SHALL maintain context across language changes
5. THE System SHALL detect the user's preferred language from their input automatically

### Requirement 8: Response Formatting

**User Story:** As a developer, I want responses to be clearly formatted and easy to understand, so that I can quickly act on the information provided.

#### Acceptance Criteria

1. WHEN the System generates a response, THE System SHALL format technical terms consistently
2. WHEN the System includes code snippets, THE System SHALL use syntax highlighting appropriate for the command type
3. WHEN the System provides multiple steps, THE System SHALL number them sequentially
4. THE System SHALL separate cultural analogies from technical details visually
5. THE System SHALL display citations as clickable links when rendered in VS Code
