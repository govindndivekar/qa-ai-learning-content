# ISTQB CT-GenAI Study Guide — Part 2: Risks, Infrastructure & Deployment

> **Based on:** ISTQB Certified Tester Testing with Generative AI (CT-GenAI) Syllabus v1.0
> **Chapters Covered:** 3, 4, and 5
> **Total Study Time:** 160 + 110 + 80 = 350 minutes (~5.8 hours)

---

## Chapter 3: Managing Risks of Generative AI in Software Testing

**Time Allocation:** 160 minutes

### Keywords to Remember

| Term | Quick Definition |
|------|-----------------|
| Security | Protection against unauthorized access and attacks |
| Vulnerability | Weakness exploitable by threats |
| Data privacy | Protection of personal/sensitive information |
| Hallucination | Wrong information created by an LLM |
| Temperature | Parameter controlling randomness/creativity of LLM outputs |
| Reasoning error | Incorrect conclusion from misinterpreted logical structures |
| Bias | Outputs favoring certain information/approaches due to training data |

---

### 3.1 Hallucinations, Reasoning Errors and Biases

#### 3.1.1 Definitions (K1)

**Learning Objective GenAI-3.1.1:** Recall the definitions of hallucinations, reasoning errors and biases in Generative AI systems.

**Hallucinations**

A hallucination occurs when an LLM generates output that appears factually incorrect or irrelevant to the given task. The output may read fluently and confidently, but the information is fabricated.

In software testing, hallucinations manifest as:
- Creating fictitious test cases referencing non-existent features or acceptance criteria
- Generating test scripts that call non-existent API endpoints or methods
- Suggesting test cases that verify acceptance criteria not present in the user story
- Inventing test frameworks or tools that do not exist

*Example 1:* You ask an LLM to generate test cases for a shopping cart user story. The LLM includes: "Verify that loyalty points are deducted correctly." The user story says nothing about loyalty points — the LLM hallucinated this requirement.

*Example 2:* The LLM generates Selenium code using `driver.findElement(By.testId("submit-btn"))` — but `By.testId` does not exist in Selenium. The LLM confidently fabricated a non-existent API method.

*Example 3:* The LLM states: "According to ISO/IEC 29119-14, AI test cases must be reviewed by two independent reviewers." This standard part does not exist.

**Reasoning Errors**

Reasoning errors occur when LLMs misinterpret logical structures — cause-and-effect relationships, conditional logic, or step-by-step problem-solving — leading to incorrect conclusions. Unlike humans who reason logically, LLMs rely on pattern matching, which breaks down with complex logic.

*Example — Test Effort Estimation:*
```
Prompt: "100 test cases. 60 take 10 min, 30 take 20 min, 10 take 45 min.
3 testers, 8 hrs/day. How many days?"

Correct: (600+600+450)=1650 min / (3×480)=1440 min per day → ~1.15 days → 2 days
LLM might answer "3 days" due to arithmetic errors or wrong division approach.
```

*Example — Test Prioritization:* Given test dependencies (TC-A before TC-B, TC-B before TC-C), an LLM might produce an execution order violating these dependencies.

**Biases**

LLM biases originate from training data imbalances. They cause outputs that favor certain types of information, approaches, or assumptions.

*Example:* An LLM trained primarily on English-language data generates test data with predominantly Western names, missing the diversity a real global user base would include (Chinese, Indian, Arabic names, names with special characters).

*Example:* When generating acceptance criteria, the LLM consistently favors functional requirements over non-functional ones (performance, accessibility, security) because functional requirements are more prevalent in training data.

**Root Causes:** All three problems stem from the nature of training data and inherent limitations of the transformer architecture.

**Comparison Table**

| Aspect | Hallucination | Reasoning Error | Bias |
|--------|--------------|-----------------|------|
| What happens | Fabricates information | Makes wrong logical conclusions | Favors certain patterns |
| Root cause | Statistical generation without factual grounding | Pattern matching, not true reasoning | Imbalanced training data |
| Detection | Cross-reference with source material | Check logic and calculations manually | Review for representation fairness |
| Testing impact | Invalid test cases, wrong test data | Wrong prioritization, incorrect estimates | Unbalanced test coverage |

---

#### 3.1.2 Identifying Hallucinations, Reasoning Errors and Biases (K3)

**Learning Objective GenAI-3.1.2:** Identify hallucinations, reasoning errors and biases in LLM output.

**Hallucination Detection:**

1. **Cross-Verification:** Compare AI output with existing documentation, requirements, and known system behavior. Create a traceability matrix mapping each generated test case to a specific requirement — any unmappable test case is potentially hallucinated.

2. **Domain Expertise Consultation:** Have subject matter experts validate generated content accuracy, especially for nuanced domain-specific aspects.

3. **Consistency Checks:** Verify outputs are consistent with each other and known information. If two generated test cases contradict each other, investigate.

**Reasoning Error Detection:**

1. **Logical Validation:** Evaluate the logical flow — check for coherence, mathematical accuracy, and correct application of dependencies and constraints.

2. **Output Testing:** Run generated test cases or scripts against actual test objects. Compilation errors, wrong expected results, or runtime failures reveal LLM reasoning errors.

**Bias Detection:**

1. **Reviewing Generated Testware:** Examine synthetic test data for fair representation — does it cover diverse demographics, languages, and edge cases?

2. **Assessing Bias in Test Types:** Check whether output underrepresents certain test types (e.g., consistently missing non-functional tests).

**Practical Exercise — Identify the Issues:**

```
Given this LLM output for "user registration" testing:

TC-01: Register with name "John Smith" and email "john@gmail.com" → Success
TC-02: Register with name "Jane Doe" and email "jane@yahoo.com" → Success  
TC-03: Register with empty name → Error message displayed
TC-04: Register with SQL injection in name → Error message
TC-05: Verify registration calculates user's credit score → Score displayed
TC-06: Register with name "Mike Johnson" and email "mike@hotmail.com" → Success
```

*Issues:*
- **Hallucination (TC-05):** Credit score calculation is not part of registration
- **Bias (TC-01, TC-02, TC-06):** All names are Western/Anglo-Saxon — missing diverse names
- **Missing coverage:** No tests for maximum length names, Unicode characters, single-character names

---

#### 3.1.3 Mitigation Techniques (K2)

**Learning Objective GenAI-3.1.3:** Summarize mitigation techniques for GenAI hallucinations, reasoning errors and biases.

Problems are more likely when prompts are poorly designed or lack relevant context.

**Key Mitigation Techniques:**

1. **Provide Complete Context:** Ensure prompts contain all relevant requirements, acceptance criteria, and domain rules.

2. **Divide Prompts into Manageable Segments:** Use prompt chaining. Verify each intermediate output before proceeding. Detects reasoning errors early.

3. **Use Clear, Interpretable Data Formats:** Avoid ambiguous formats. Use tables, numbered lists, and clear headings.

4. **Select Appropriate Model:** Use reasoning models for logical tasks, instruction-tuned models for straightforward generation.

5. **Compare Results Across Models:** Evaluate the same prompt across multiple LLMs and compare outputs.

**Additional Techniques (Chapter 4):** RAG (grounds responses in actual data) and Fine-Tuning (adapts model to domain-specific patterns).

---

#### 3.1.4 Mitigating Non-Deterministic Behavior (K1)

**Learning Objective GenAI-3.1.4:** Recall mitigation techniques for non-deterministic behavior of LLMs.

Non-deterministic behavior means output varies even with identical inputs, caused by probabilistic sampling during inference.

**Strategy 1: Adjust Temperature Parameter**

```
Temperature 0.0 (Low):  Most probable token always selected → Deterministic, repetitive
Temperature 0.7 (Medium): Balanced creativity and consistency
Temperature 1.0+ (High): Broad sampling → Creative but unpredictable
```

Lower temperature reduces randomness but limits creativity. Higher temperature enables exploration but reduces reproducibility.

**Strategy 2: Set Random Seeds**

Some LLM implementations allow a seed value for the random number generator, producing the same pseudo-random sequence and improving reproducibility. Not available in all platforms.

**Key Takeaway:** Complete reproducibility cannot be guaranteed. Build processes to account for variability.

---

### 3.2 Data Privacy and Security Risks

#### 3.2.1 Key Risks (K2)

**Learning Objective GenAI-3.2.1:** Explain key data privacy and security risks.

**Data Privacy Concerns:**
- **Unintentional Data Exposure:** AI may generate outputs revealing sensitive information from training data or prompts
- **Lack of Control Over Data Usage:** Tools may store/process sensitive data without consent, leading to unauthorized access
- **Compliance Risks:** Non-compliance with GDPR and other regulations could lead to legal disputes

**Security Risks:**
- **Infrastructure Vulnerability:** LLM-powered test infrastructure susceptible to data breaches
- **Malicious Exploitation:** Attackers exploit LLM vulnerabilities to alter behavior or extract data
- **Malicious Input Data:** Deliberately introduced data that misleads LLMs

#### 3.2.2 Vulnerabilities in GenAI Testing (K2)

**Learning Objective GenAI-3.2.2:** Give examples of data privacy and vulnerabilities.

| Attack Vector | Description | Testing Example |
|---------------|-------------|-----------------|
| **Data Exfiltration** | Requests designed to extract confidential training data | Long prompts overloading context window, causing revelation of training data snippets |
| **Request Manipulation** | Data disrupting AI output | Images with hidden instructions provoking hallucinations on acceptance criteria |
| **Data Poisoning** | Manipulating training data | Fake evaluations when rating AI-generated test reports |
| **Malicious Code Generation** | LLM generates backdoors | Code opening communication channel with a malicious IP |

#### 3.2.3 Mitigation Strategies (K2)

**Learning Objective GenAI-3.2.3:** Summarize mitigation strategies for data privacy and security.

**Data Privacy Measures:**
1. **Data Minimization:** Use only necessary non-sensitive data
2. **Anonymization/Pseudonymization:** Mask sensitive information
3. **Secure Storage and Transmission:** Strong encryption and access controls
4. **Training Programs:** Policies ensuring responsible GenAI use

**Security Strategies:**
5. **Systematic Review** of generated output by humans
6. **Cross-LLM Evaluation:** Compare outputs across multiple LLMs
7. **Secure Environment Selection:**
   - Option 1: Commercial secure offering from LLM provider
   - Option 2: LLM in secure cloud
   - Option 3: LLM on organization's own infrastructure
8. **Regular Security Audits**
9. **Stay Updated** with latest security practices

**Recommendation:** Involve Security Engineers, Legal counsel, CTO, or CISO in deployment decisions.

---

### 3.3 Energy Consumption and Environmental Impact

#### 3.3.1 Energy Impact (K2)

**Learning Objective GenAI-3.3.1:** Explain the impact on energy consumption.

Research shows LLM training and processing require intensive computing resources. Generating a single image consumes energy equivalent to fully charging a smartphone; generating text consumes only a small fraction. The cumulative effect of millions of users creates substantial environmental strain.

**Best Practices:**
- Refine prompts before sending (avoid trial-and-error)
- Use SLMs for simple tasks, LLMs only when needed
- Batch related queries
- Cache and reuse previous results

---

### 3.4 AI Regulations, Standards, and Frameworks

#### 3.4.1 Key Regulations and Standards (K1)

**Learning Objective GenAI-3.4.1:** Recall examples of AI regulations, standards and frameworks.

| Name | Type | Description | Testing Application |
|------|------|-------------|-------------------|
| **ISO/IEC 42001:2023** | Standard | AI management system requirements | Consistency and reliability in GenAI testing |
| **ISO/IEC 23053:2022** | Standard | AI lifecycle framework | Data quality, transparency, fault tolerance |
| **EU AI Act** | Regulation | Risk-based AI classification | Transparency, accountability, bias mitigation |
| **NIST AI RMF** | Framework | AI risk management guidelines | Fairness and risk prevention in testing |

Test organizations must stay updated as regulatory landscapes evolve.

---

## Chapter 4: LLM-Powered Test Infrastructure

**Time Allocation:** 110 minutes

### Keywords to Remember

| Term | Definition |
|------|-----------|
| Fine-tuning | Adapting a pre-trained LLM to specific tasks using targeted datasets |
| LLM-powered agent | Application integrating LLM reasoning with tools to perform tasks |
| LLMOps | Practices for deploying, monitoring, and maintaining LLMs in production |
| RAG | Combining LLM with retrieved data for more accurate responses |
| Vector database | Database optimized for storing high-dimensional vector representations |

---

### 4.1 Architectural Approaches

#### 4.1.1 Key Components (K2)

**Learning Objective GenAI-4.1.1:** Explain key architectural components of LLM-powered test infrastructure.

LLM-powered test infrastructure integrates LLMs into the testing process for enhanced automation, reasoning, and decision-making.

**Architecture:**
```
┌───────────────────────────────────┐
│           FRONT-END               │
│  (Tester Interface)               │
└──────────────┬────────────────────┘
               │
┌──────────────▼────────────────────┐
│           BACK-END                │
│  Authentication │ Prompt Prep     │
│  Data Retrieval │ Post-Processing │
└──────┬────────────────┬───────────┘
       │                │
┌──────▼──────┐  ┌──────▼──────────┐
│  External   │  │  Integrated     │
│  Data       │  │  LLM            │
│  - Relational│  │  (API or        │
│  - Vector DB │  │   in-house)     │
└─────────────┘  └─────────────────┘
```

**Four Distinguishing Features:**
1. **LLM as Smart Processing:** Interprets and reasons about test products, not just retrieves data
2. **Dynamic Content Generation:** Generates insights from context (vs. scripted responses)
3. **Multi-Source Back Ends:** Relational DBs (structured data) + Vector DBs (semantic retrieval)
4. **Post-Processing Enhancement:** Back-end refines raw LLM output for test process alignment

#### 4.1.2 Retrieval-Augmented Generation (K2)

**Learning Objective GenAI-4.1.2:** Summarize Retrieval-Augmented Generation.

RAG enhances LLMs by incorporating additional data sources, increasing relevance and accuracy.

**Preprocessing Phase:**
1. Break large documents into chunks (256-512 tokens)
2. Clean and process chunks
3. Encode into embeddings (high-dimensional vectors)
4. Store in vector database

**Runtime Phase:**
1. **Retrieval:** Given a user query, retrieve relevant chunks from vector DB based on semantic similarity between query embedding and stored chunk embeddings
2. **Generation:** Feed retrieved context + query to LLM, which generates a response combining its knowledge with the newly acquired data

**With vs Without RAG:**

| Without RAG | With RAG |
|-------------|----------|
| Generic test cases based on common patterns | Test cases specific to YOUR application's business rules |
| May use wrong timeouts, password rules, etc. | Uses your actual requirements (e.g., 2-hour expiry, 12-char passwords) |
| No knowledge of your project | Accesses latest specifications, requirements, and existing test data |

**Testing Application:** Enables real-time access to enterprise data — databases, documentation, repositories — ensuring test tasks align with current project state.

#### 4.1.3 LLM-Powered Agents (K2)

**Learning Objective GenAI-4.1.3:** Explain the role of LLM-powered agents in automating test processes.

LLM-powered agents are specialized applications for semi-autonomous or autonomous task processing. Unlike chatbots (question-response only), agents can invoke tools to interact with external systems.

**Autonomy Levels:**
- **Autonomous:** Minimal human intervention (e.g., agent monitors results, creates defect reports, assigns them)
- **Semi-Autonomous:** Periodic human oversight (e.g., agent generates test cases for human review)

**Multi-Agent Architectures:** Multiple specialized agents coordinate — one analyzes requirements, another generates tests, a third writes scripts, an orchestrator coordinates.

**Limitation:** Same problems as LLMs — hallucinations, reasoning errors, biases. Mitigate with automated verification and semi-autonomous design for critical tasks.

---

### 4.2 Fine-Tuning and LLMOps

#### 4.2.1 Fine-Tuning (K2)

**Learning Objective GenAI-4.2.1:** Explain fine-tuning of language models for test tasks.

Fine-tuning adapts a pre-trained model to a specific domain by further training on targeted data.

**When Suitable:** Domain-specific reasoning, organization-specific vocabulary, consistent output formats. Can apply to both LLMs and SLMs (SLMs are less resource-intensive).

**Example:** Fine-tune on your organization's user stories and test cases. Model learns your format, terminology, and conventions.

**Challenges:**
1. Biased/inaccurate results from poor training data
2. Overfitting — too specialized, poor on new scenarios
3. Opacity — unclear decision-making
4. Computational resource demands

#### 4.2.2 LLMOps (K2)

**Learning Objective GenAI-4.2.2:** Explain LLMOps and its role in deploying LLMs for testing.

Three approaches (NOT mutually exclusive):

| Approach | Considerations | Best For |
|----------|---------------|----------|
| **AI Chatbot** | Privacy/security, cost. Use LLM-as-a-Service or deploy open-source in-house | Quick adoption, individual productivity |
| **Test Tool with GenAI** | Evaluate vendor assurances. Complements existing processes | Integrated team solutions |
| **In-House Development** | Maximum data control. Requires LLM infrastructure expertise | Custom requirements, maximum security |

---

## Chapter 5: Deploying and Integrating GenAI in Test Organizations

**Time Allocation:** 80 minutes

### 5.1 Roadmap for Adoption

#### 5.1.1 Shadow AI Risks (K1)

**Learning Objective GenAI-5.1.1:** Recall the risks of shadow AI.

**Shadow AI:** Use of GenAI tools without formal organizational approval.

**Risks:**
1. **Security/Privacy Weaknesses:** Personal tools lack enterprise security
2. **Compliance/Regulatory Issues:** Non-compliance with standards
3. **Intellectual Property Disputes:** Unclear licensing for copyrighted data

#### 5.1.2 GenAI Strategy (K2)

**Learning Objective GenAI-5.1.2:** Explain key aspects of a GenAI strategy for testing.

Six foundational elements: Test Objectives (measurable, aligned with business goals), LLM Selection (compatible with infrastructure), Data Quality (accurate, secure inputs), Training Programs (technical and ethical skills), Measurement (metrics per Section 2.3.1), and Compliance/Ethics (rules for sensitive data, transparency, quality gates).

#### 5.1.3 LLM Selection Criteria (K2)

**Learning Objective GenAI-5.1.3:** Summarize key criteria for selecting LLMs/SLMs.

Four criteria: Model Performance (benchmark on your tasks), Fine-Tuning Potential (domain adaptation feasibility), Recurring Cost (licensing + operational), Community and Support (documentation, forums).

#### 5.1.4 Adoption Phases (K1)

**Learning Objective GenAI-5.1.4:** Recall key adoption phases.

1. **Discovery:** Awareness building, experimenting with use cases
2. **Initiation:** Prioritizing use cases, developing expertise
3. **Utilization:** Full integration, continuous monitoring, sustainable scaling

Phases can run in parallel for different use cases. Address job displacement concerns early.

### 5.2 Managing Change

#### 5.2.1 Essential Skills (K2)

Testers need: prompt engineering, context window understanding, AI output review methods, domain+AI skill integration, data sanitization practices, environmental awareness.

#### 5.2.2 Cultivating AI Skills (K1)

Hands-on practice, structured learning paths, prompt pattern libraries, internal communities of practice.

#### 5.2.3 Evolving Roles (K1)

**Testers:** From test specialists → AI-assisted specialists (reviewing AI output, prompt refinement, maintaining prompt libraries).

**Test Managers:** Add AI strategy, AI risk management, governance frameworks, hybrid human-AI team leadership.

---

## Complete Glossary — All 36 GenAI-Specific Terms

| # | Term | Definition |
|---|------|-----------|
| 1 | AI chatbot | Conversational agent using LLMs for interactive communication |
| 2 | Context window | Span of text (tokens) a model considers when generating responses |
| 3 | Deep learning | ML using neural networks with multiple layers |
| 4 | Embedding | Dense vector representing token's semantic/syntactic relationships |
| 5 | Feature | Measurable attribute of input data used by ML |
| 6 | Few-shot prompting | Giving a model examples in the prompt to guide responses |
| 7 | Fine-tuning | Adapting a pre-trained LLM using labeled examples for specific tasks |
| 8 | Foundation LLM | General-purpose model pre-trained on diverse text data |
| 9 | Generative AI | AI using ML models to generate new content resembling human work |
| 10 | GPT | Transformer-based deep learning model pre-trained on vast text |
| 11 | Hallucination | Wrong information created by an LLM |
| 12 | Instruction-tuned LLM | Foundation model trained to follow instructions via feedback |
| 13 | LLM | Program using large language data to understand and produce text |
| 14 | LLM-powered agent | Application integrating LLM reasoning with tools for task execution |
| 15 | LLMOps | Practices for deploying/monitoring/maintaining LLMs in production |
| 16 | Machine learning | Systems learning from data using computational techniques |
| 17 | Meta prompting | Higher-level instructions that generate specific prompts |
| 18 | Multimodal model | GenAI processing multiple data types (text, images, audio) |
| 19 | NLP | Computer processing of natural language data |
| 20 | One-shot prompting | Prompt with one example guiding the LLM |
| 21 | Prompt | Natural language input to elicit a specific AI response |
| 22 | Prompt chaining | Using one prompt's output as the next prompt's input |
| 23 | Prompt engineering | Designing/refining prompts for desired LLM outputs |
| 24 | Reasoning LLM | LLM emphasizing logical inference and multi-step reasoning |
| 25 | RAG | Combining LLM with retrieved relevant data for accurate responses |
| 26 | Shadow AI | GenAI use without formal organizational approval |
| 27 | SLM | Small language model balancing efficiency and task-specific understanding |
| 28 | Symbolic AI | AI using symbols, rules, and structured knowledge for reasoning |
| 29 | System prompt | Hidden instruction set establishing LLM context/tone/boundaries |
| 30 | Temperature | Parameter controlling randomness of LLM outputs |
| 31 | Tokenization | Breaking text into smaller units for model processing |
| 32 | Transformer | Architecture using self-attention for long-range dependencies |
| 33 | User prompt | User-entered instruction directing the LLM's response |
| 34 | Vector database | Database for storing high-dimensional vector representations |
| 35 | Vision-language model | GenAI jointly processing visual and textual data |
| 36 | Zero-shot prompting | Prompt with no examples, relying on model's existing knowledge |

---

## Practice Exercises — Chapters 3-5

**Q1 (K1).** A hallucination in GenAI is:
A) The LLM produces biased output  B) The LLM generates wrong information  C) The LLM gives varied outputs  D) The LLM refuses to answer

**Q2 (K2).** An LLM estimates "5 days" test effort but the correct answer is "8 days." This is:
A) Hallucination  B) Reasoning error  C) Bias  D) Non-determinism

**Q3 (K1).** Which parameter reduces LLM output randomness?
A) Context window  B) Temperature  C) Embedding dimension  D) Token limit

**Q4 (K2).** Which attack manipulates training data?
A) Data exfiltration  B) Request manipulation  C) Data poisoning  D) Malicious code generation

**Q5 (K2).** Maximum data privacy control requires:
A) Commercial LLM offering  B) Secure cloud LLM  C) On-premises LLM  D) Public chatbot

**Q6 (K2).** In RAG, the vector database stores:
A) User prompts  B) Embedded document chunks  C) LLM weights  D) Test results

**Q7 (K2).** Key difference between agents and chatbots:
A) Speed  B) Agents invoke external tools  C) Chatbots are more accurate  D) Agents don't use LLMs

**Q8 (K1).** Shadow AI is:
A) AI monitoring other AI  B) Unapproved GenAI use  C) Dark mode AI  D) Backup AI systems

**Q9 (K2).** Which is NOT a fine-tuning challenge?
A) Overfitting  B) High compute cost  C) Increased context window  D) Opacity

**Q10 (K2).** Full GenAI integration occurs in which phase?
A) Discovery  B) Initiation  C) Utilization and Iteration  D) Planning

**Q11 (K3).** LLM generates only Western European names for test data. This is:
A) Hallucination  B) Reasoning error  C) Bias  D) Non-determinism

**Q12 (K2).** Which technique grounds LLM responses in company data?
A) Temperature adjustment  B) Fine-tuning  C) RAG  D) Random seeds

**Q13 (K2).** Best way to detect reasoning errors in LLM test plans:
A) Grammar check  B) Check output length  C) Manually verify logic  D) Rerun prompt

**Q14 (K1).** Which regulation classifies AI by risk level?
A) ISO/IEC 42001  B) NIST AI RMF  C) EU AI Act  D) ISO/IEC 23053

**Q15 (K2).** Data minimization is important because:
A) Reduces costs  B) Speeds responses  C) Reduces privacy risks  D) Improves accuracy

**Answer Key:** Q1:B | Q2:B | Q3:B | Q4:C | Q5:C | Q6:B | Q7:B | Q8:B | Q9:C | Q10:C | Q11:C | Q12:C | Q13:C | Q14:C | Q15:C

---

## Additional Topics: Emerging AI Risks and Governance

### Deepfakes and Synthetic Media
Testers must be aware of deepfake risks when testing media-processing systems. Test strategies should include adversarial testing with synthetic media.

### AI Supply Chain Security
Using third-party models means inheriting their security posture. Evaluate model provenance, training data integrity, and API security.

### Responsible AI Frameworks Beyond the EU
- US Executive Order on AI (2023): Safety, security, trustworthiness
- China's AI Regulations: Algorithmic impact assessments
- OECD AI Principles: International responsible AI framework

### Carbon Footprint Awareness
Tools like ML CO2 Impact Calculator and CodeCarbon estimate AI environmental impact. Include environmental metrics in test strategy.

---

## Index of Key Terms

| Term | Section |
|------|---------|
| Adoption phases | 5.1.4 |
| Anonymization | 3.2.3 |
| Bias | 3.1.1 |
| Data exfiltration | 3.2.2 |
| Data minimization | 3.2.3 |
| Data poisoning | 3.2.2 |
| Energy consumption | 3.3.1 |
| EU AI Act | 3.4.1 |
| Fine-tuning | 4.2.1 |
| GDPR | 3.2.1 |
| Hallucination | 3.1.1 |
| ISO/IEC 42001 | 3.4.1 |
| LLM-powered agent | 4.1.3 |
| LLMOps | 4.2.2 |
| Multi-agent architecture | 4.1.3 |
| NIST AI RMF | 3.4.1 |
| Non-deterministic behavior | 3.1.4 |
| RAG | 4.1.2 |
| Random seeds | 3.1.4 |
| Reasoning error | 3.1.1 |
| Shadow AI | 5.1.1 |
| Temperature | 3.1.4 |
| Vector database | 4.1.2 |
| Vulnerability | 3.2.1 |

## References

1. ISTQB CT-GenAI Syllabus v1.0 (2025)
2. ISO/IEC 42001:2023 — AI Management System
3. ISO/IEC 23053:2022 — Framework for AI Systems Using ML
4. EU AI Act — Regulation (EU) 2024/1689
5. NIST AI Risk Management Framework (AI RMF 1.0)
6. GDPR — Regulation (EU) 2016/679
7. Luccioni, A.S. et al. (2024a). "Power Hungry Processing"
8. Berthelot, A. et al. (2024). "Environmental impact of Generative AI"
9. Zhao, P. et al. (2024). "Retrieval-Augmented Generation for AI-Generated Content"
10. Wang, L. et al. (2024). "Survey on LLM-based Autonomous Agents"

---

*End of CT-GenAI Study Guide. Continue to CT-AI Study Guides for AI Testing certification.*
