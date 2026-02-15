## Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    actor Developer as "Developer (Hinglish)"
    
    box "Interface" #e3f2fd
        participant VSCode as "VS Code Ext"
        participant Pulse as "Pulse-Batch"
    end

    box "The Brain (AI)" #fff3e0
        participant Orch as "Bedrock Orch"
        participant RAG as "Gyan-Setu (KB)"
        participant Guard as "Karma-Kavach"
        participant Drishti as "Dosh-Drishti"
    end

    box "Execution Engine" #e8f5e9
        participant Local as "Local Exec (Git/Shell)"
        participant SSM as "AWS SSM (Remote)"
        participant AWS as "AWS Cloud"
    end

    note over Developer,AWS: === SCENARIO 1: INITIALIZATION ===
    Developer->>VSCode: Open Dev-Sarathi
    VSCode->>AWS: Authenticate (IAM Role)
    AWS-->>VSCode: Session Active

    note over Developer,AWS: === SCENARIO 2: VERNACULAR LEARNING (Gyan-Setu) ===
    Developer->>VSCode: "Bhai, S3 storage kya hota hai?"
    VSCode->>Pulse: Stream Audio
    Pulse->>Orch: Transcribed Text: "What is S3?"
    Orch->>RAG: Fetch "AWS S3 Concepts"
    RAG-->>Orch: Returns Docs + Analogies
    Orch-->>Developer: "S3 ek digital godown ki tarah hai..." (Vernacular)

    note over Developer,AWS: === SCENARIO 3: CODE GENERATION (Vani-Srijan) ===
    Developer->>VSCode: "Python mein Fibonacci series ka function likho"
    VSCode->>Orch: Intent: CODE_GEN (Python)
    Orch->>Orch: Generate Code Snippet
    Orch-->>Developer: Returns Python Code Block
    Developer->>VSCode: "Insert Code" (Click)

    note over Developer,AWS: === SCENARIO 4: LOCAL AUTOMATION (Vani-Srijan) ===
    Developer->>VSCode: "Code change commit kardo"
    VSCode->>Orch: Intent: GIT_COMMIT
    Orch->>Guard: Safety Check
    Guard-->>Orch: Safe (Allow)
    Orch->>Local: git commit -m "Update logic"
    Local-->>Developer: "Commit Successful: [a1b2c3]"

    note over Developer,AWS: === SCENARIO 5: SAFETY INTERCEPTION (Karma-Kavach) ===
    Developer->>VSCode: "Production DB uda do" (Delete DB)
    VSCode->>Orch: Intent: DELETE_RDS
    Orch->>Guard: Evaluate Risk
    Guard-->>Orch: BLOCK (High Risk)
    Orch-->>Developer: "Warning: Destructive Action hoga! Override?"
    Developer->>Orch: "Cancel" (Action Aborted)

    note over Developer,AWS: === SCENARIO 6: ERROR DIAGNOSTICS (Dosh-Drishti) ===
    Developer->>VSCode: "Naya EC2 launch karo"
    VSCode->>Orch: Intent: LAUNCH_INSTANCE
    Orch->>SSM: aws ec2 run-instances ...
    SSM->>AWS: Execute Command
    AWS-->>SSM: Error: "InsufficientCapacity"
    SSM-->>Orch: Execution Failed (Exit Code 1)
    
    Orch->>Drishti: Analyze Error Logs
    Drishti->>Drishti: Reason: Region Outage / Limit Reached
    Drishti-->>Developer: "Error: Capacity nahi hai. Aap 'ap-south-1a' try kar sakte hain."
