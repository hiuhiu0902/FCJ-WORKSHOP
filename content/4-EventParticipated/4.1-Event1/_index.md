---
title: "AWS Cloud Mastery Series #1"
date: "2025-11-15T08:30:00+07:00"
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# Summary Report: “AI/ML/GenAI on AWS Workshop”

### Event Overview
**Date:** November 15, 2025
**Location:** AWS Vietnam Office (Bitexco Financial Tower)
**Scale:** Over 200 attendees, including students and AWS Cloud Club members.

### Event Objectives
- Provide an overview of the AI/ML landscape in Vietnam.
- Deep dive into **Amazon SageMaker** for end-to-end Machine Learning.
- Explore **Generative AI** capabilities with **Amazon Bedrock**.
- Learn practical strategies for building scalable AI Agents and implementing RAG (Retrieval-Augmented Generation).

### Speakers
- **Kha Van**
- **Kiet Lam**
- **Aiden Dinh**

### Key Highlights

#### 1. AWS AI/ML Services Overview (Traditional ML)
- **Amazon SageMaker:** Introduced as a comprehensive platform for the entire ML lifecycle.
- **Workflow:** Covered data preparation/labeling, model training, tuning, and final deployment.
- **MLOps:** Emphasized the importance of integrated MLOps capabilities to streamline operations.
- **Live Demo:** A walkthrough of SageMaker Studio to visualize the environment.

#### 2. Generative AI with Amazon Bedrock
- **Foundation Models (FMs):** Comparison and selection guide for top models like **Claude, Llama, and Titan**.
- **Prompt Engineering:** Advanced techniques including Chain-of-Thought reasoning and Few-shot learning to improve model output.
- **RAG (Retrieval-Augmented Generation):** Architecture breakdown on how to connect FMs with proprietary knowledge bases (Vector Databases) to minimize hallucinations.
- **Bedrock Agents:** Building multi-step workflows where AI can execute tasks (API calls) rather than just chatting.
- **Guardrails:** Implementing safety filters to block harmful content.

#### 3. From PoC to Production
- The session highlighted the journey from a Proof-of-Concept (PoC) to a production-ready solution.
- Focus on **Amazon Bedrock AgentCore** strategies for building scalable solutions.

### Key Takeaways

#### The Power of Choice
- With Amazon Bedrock, we aren't locked into one model. We can choose the right tool for the job (e.g., Claude for reasoning, Titan for embeddings).

#### RAG is Essential
- To build reliable enterprise apps, we cannot rely solely on the model's training data. RAG is the key to providing accurate, up-to-date context.

#### Security in AI
- **Guardrails** are not optional. Configuring content filtering is crucial before releasing any GenAI application to the public.

### Applying to Work

- **Experiment with Bedrock:** I plan to build a simple chatbot using Amazon Bedrock to test different prompts.
- **Implement RAG:** I will attempt to integrate a small knowledge base (e.g., product documentation) to see how RAG improves answers compared to a standard prompt.
- **Explore Agents:** Investigate how Bedrock Agents can be used to automate backend tasks in my Game Card project (e.g., checking order status via natural language).

### Event Experience

The atmosphere at the AWS Vietnam Office was enthusiastic, with over **200 attendees**.

**What stood out:**
- **Comprehensive Content:** The transition from traditional ML (SageMaker) to modern GenAI (Bedrock) provided a complete picture of the AWS ecosystem.
- **Engaging Demos:** The live demo of building a GenAI chatbot made the abstract concepts concrete.
- **Community Spirit:** The Kahoot mini-game at the end was a fun way to test knowledge and bond with the community.
- **Mentorship:** A special thanks to our mentor, **Mr. Nguyen Gia Hung**, for making this "First Cloud AI Journey" possible.

> **Conclusion:** This workshop bridged the gap between theory and practice, giving me the confidence to start building AI-powered features for my projects.