# ISTQB CT-GenAI Study Guide — Part 1: Foundations & Prompt Engineering

> **Based on:** ISTQB Certified Tester Testing with Generative AI (CT-GenAI) Syllabus v1.0
> **Chapters Covered:** 1 and 2
> **Total Study Time:** 100 + 365 = 465 minutes (~7.75 hours)

---

## Exam Overview

The CT-GenAI certification requires the ISTQB Foundation Level (CTFL) as a prerequisite. The exam tests knowledge at three cognitive levels:

- **K1 (Remember):** Recall definitions and concepts
- **K2 (Understand):** Explain, compare, give examples, summarize
- **K3 (Apply):** Use a technique or procedure in a given context

Hands-on Objectives (HO) levels: H0 (demo/video), H1 (guided exercise), H2 (exercise with hints).

### Business Outcomes

Upon certification, a candidate can:

- **GenAI-BO1:** Understand fundamental concepts, capabilities, and limitations of generative AI
- **GenAI-BO2:** Develop practical skills in prompting LLMs for software testing
- **GenAI-BO3:** Gain insight into risks and mitigations of using GenAI for testing
- **GenAI-BO4:** Gain insight into applications of GenAI solutions for testing
- **GenAI-BO5:** Contribute to the definition and implementation of a GenAI strategy and roadmap for testing

---

## Chapter 1: Introduction to Generative AI for Software Testing

**Time Allocation:** 100 minutes

### Keywords to Remember

You must be able to recall and recognize these terms:

| Term | Quick Definition |
|------|-----------------|
| AI chatbot | Conversational agent using LLMs for interactive communication |
| Context window | Span of text (in tokens) an LLM considers when generating a response |
| Deep learning | ML using neural networks with multiple layers |
| Embedding | Numerical vector representing a token's meaning and relationships |
| Feature | Individual measurable attribute of input data used by ML |
| Foundation LLM | General-purpose model pre-trained on vast, diverse datasets |
| Generative AI | AI that creates new content (text, images, code) by learning patterns |
| Generative pre-trained transformer (GPT) | Transformer-based deep learning model pre-trained on vast text data |
| Instruction-tuned LLM | Foundation model fine-tuned to follow human instructions |
| Large language model (LLM) | Model trained on large textual datasets to understand and produce text |
| Machine learning | Systems that learn from data or experience using computational techniques |
| Multimodal model | GenAI model processing multiple data types (text, images, audio) |
| Reasoning LLM | LLM emphasizing logical inference and multi-step problem-solving |
| Symbolic AI | AI using symbols, rules, and structured knowledge for reasoning |
| Tokenization | Breaking text into smaller units (tokens) for LLM processing |
| Transformer | Architecture using self-attention to capture long-range text dependencies |

---

### 1.1 Generative AI Foundations and Key Concepts

#### 1.1.1 Types of AI: Symbolic AI, Classical ML, Deep Learning, and Generative AI (K1)

**Learning Objective GenAI-1.1.1:** Recall different types of AI.

Artificial intelligence encompasses multiple approaches, each with distinct characteristics and applications in software testing. Understanding these types helps testers select the right approach for different testing challenges.

**Symbolic AI** is a rule-based approach that mimics human decision-making using symbols and logical rules. Think of it as a sophisticated decision tree: "IF the bug report contains the word 'crash' AND the module is 'payment', THEN assign priority P1." Symbolic AI systems are entirely transparent — you can trace every decision back to a specific rule. However, they struggle with ambiguity, require extensive manual rule-crafting, and cannot learn from data.

*Testing Example:* A rule engine that automatically categorizes incoming bug reports based on keywords and assigns them to the appropriate team. The rules are hand-coded: defects mentioning "UI" go to the frontend team, those mentioning "database" go to the backend team.

**Classical Machine Learning** takes a data-driven approach. Instead of writing rules, you provide data and let the algorithm discover patterns. This requires careful data preparation, feature selection (choosing which attributes matter), and model training. The model learns statistical relationships in the data and uses them for predictions.

*Testing Example:* A defect prediction model trained on historical project data — lines of code changed, developer experience, code complexity, module age — that predicts which modules are most likely to contain defects. The tester manually selects which features (attributes) to include.

**Deep Learning** uses neural networks with multiple hidden layers to automatically learn features from data. Unlike classical ML, you do not need to manually define which features matter — the network discovers relevant patterns on its own. Deep learning excels with large, complex datasets such as images, video, audio, and text, though human involvement in annotation, tuning, and validation may still be required.

*Testing Example:* An image-based testing tool that uses a convolutional neural network to compare screenshots of a web application against reference images, automatically detecting visual regressions — misaligned buttons, missing icons, color changes — without needing hand-coded comparison rules.

**Generative AI** uses deep learning techniques to create entirely new content — text, images, code, music — by learning and mimicking patterns from training data. Models like LLMs can generate text, write code, and simulate reasoning and problem-solving within the scope of their training. The key advantage for testing is that GenAI uses pre-trained models applicable directly to test tasks without an additional training phase, although this carries risks (covered in Chapter 3).

*Testing Example:* Given a user story for a login feature, an LLM generates a complete set of test cases covering positive flows, negative flows, boundary values, and edge cases — in seconds rather than hours.

**Comparison Table: Types of AI**

| Dimension | Symbolic AI | Classical ML | Deep Learning | Generative AI |
|-----------|------------|-------------|---------------|---------------|
| Approach | Rule-based | Data-driven | Neural networks | Deep learning + generation |
| Learning | No learning; hand-coded rules | Learns from labeled/unlabeled data | Learns features automatically | Learns patterns for content creation |
| Feature Engineering | Not applicable | Manual feature selection required | Automatic feature extraction | Automatic |
| Transparency | Fully transparent | Moderately interpretable | Black-box | Black-box |
| Data Needs | Rules, not data | Moderate datasets | Large datasets | Massive datasets for pre-training |
| Training Required by Tester | None (uses existing rules) | Yes (train for specific task) | Yes (train or fine-tune) | No (uses pre-trained models) |
| Testing Application | Bug categorization rules | Defect prediction | Visual regression testing | Test case generation, analysis |
| Strengths | Deterministic, explainable | Good with structured data | Excellent with complex data | Versatile, no extra training needed |
| Limitations | Rigid, no learning | Manual feature work | Needs large data, opaque | Hallucinations, non-determinism |

---

#### 1.1.2 Basics of Generative AI and Large Language Models (K2)

**Learning Objective GenAI-1.1.2:** Explain the basics of Generative AI and Large Language Models.

**Generative AI (GenAI)** is a branch of artificial intelligence that uses large, pre-trained models to generate human-like output including text, images, and code. At the heart of most text-based GenAI are Large Language Models.

**Large Language Models (LLMs)** are GenAI models that have been pre-trained on enormous textual datasets. They determine context from user prompts and produce relevant responses. Their power comes from the sheer scale of data they have consumed and the sophisticated architecture that processes it.

**Small Language Models (SLMs)** are more compact models with fewer parameters than LLMs. They are designed to provide lightweight, focused GenAI solutions — trading some capability for speed, lower cost, and easier deployment. Think of an SLM as a specialist doctor versus an LLM as a general hospital: the specialist handles specific cases efficiently, while the hospital covers everything but at greater cost.

There are five key concepts you need to understand about how LLMs work:

**1. Tokenization**

Tokenization is the process of breaking down text into smaller units called tokens. Tokens can be characters, sub-words, or whole words — the exact granularity depends on the model's tokenizer.

*Worked Example:* Consider the sentence "The test case failed unexpectedly."

A typical tokenizer (like GPT's BPE tokenizer) might break this into:
```
["The", " test", " case", " failed", " unexpected", "ly", "."]
```
That is 7 tokens. Notice that "unexpectedly" is split into "unexpected" and "ly" because the tokenizer breaks uncommon words into known sub-word units. Common words like "The" and "test" remain whole.

Why does this matter for testers? Because LLMs have a finite context window measured in tokens. If you are feeding a 200-page requirements document into an LLM, the token count determines whether it fits. A rough rule of thumb is that 1 token is approximately 0.75 English words, so 100 words translate to roughly 130-140 tokens.

**2. Embeddings**

Once text is tokenized, each token is transformed into a numerical representation called an embedding — a vector in a high-dimensional space. These vectors encode semantic, syntactic, and contextual relationships.

*Analogy:* Imagine a city map where every word has a location. Words with similar meanings are placed close together: "defect," "bug," and "fault" would be neighbors. "Performance" and "speed" would be nearby. "Test case" and "scenario" would be close. But "test case" and "banana" would be far apart. Embeddings work like this but in hundreds of dimensions, capturing subtle relationships that a two-dimensional map cannot.

This is what enables LLMs to understand that "the test failed" and "the test did not pass" mean the same thing — their embeddings are close in vector space.

**3. Transformer Architecture**

The transformer is the neural network architecture that powers modern LLMs. It excels at language tasks because it can process the context of extensive text sequences and learn how tokens relate to each other through a mechanism called "self-attention."

During inference (generating a response), the transformer predicts the next token in a sequence by leveraging all the relationships it learned during training. It generates text that is *statistically plausible* based on training data and the prompt.

**Critical caveat for testers: Plausible does NOT equal correct.** An LLM can generate a test case that reads perfectly, follows proper formatting, and sounds authoritative — but contains factually wrong expected results. This is the fundamental challenge of using GenAI in testing.

**4. Non-Deterministic Behavior**

LLMs exhibit inherent randomness due to the probabilistic nature of their inference mechanisms and hyper-parameter settings. This means that even with identical inputs, the output may vary between runs. This is fundamentally different from traditional software testing where the same input always produces the same output.

*Example:* You ask an LLM "Generate 3 test cases for a login form" twice in a row. The first time you might get test cases about valid login, empty password, and SQL injection. The second time you might get valid login, wrong password, and account lockout. Both are reasonable, but they are different.

**5. Context Window**

The context window is the amount of preceding text (measured in tokens) that the model considers when generating responses. Think of it as the model's "working memory."

A larger context window allows the model to maintain coherence over longer passages — for example, analyzing an entire large test log or a complete requirements document. However, increasing the number of tokens increases computational complexity and processing time.

*Practical implication:* If you are analyzing test results from a 50-page test report, the context window determines whether the LLM can consider the entire report at once or whether you need to break it into smaller chunks and process them separately. Modern models like GPT-4 support 128K tokens (roughly 96,000 words), while older models might only handle 4K tokens.

**Hands-On Objective HO-1.1.2 (H1): Practice Tokenization and Token Count Evaluation**

*Exercise:* Use an online tokenizer tool (such as OpenAI's Tokenizer at platform.openai.com/tokenizer) to:
1. Tokenize the following test-related texts and count the tokens:
   - "Verify that the user can log in with valid credentials"
   - "Given a registered user, When they enter correct username and password, Then they should see the dashboard"
   - A paragraph from a requirements document you are working with
2. Compare the token counts. How does the Gherkin format affect token usage?
3. If your LLM has a 4K token context window, estimate how many test cases you can include in a single prompt.

---

#### 1.1.3 Foundation, Instruction-Tuned, and Reasoning LLMs (K2)

**Learning Objective GenAI-1.1.3:** Distinguish between Foundation, Instruction-Tuned and Reasoning LLMs.

LLMs come in three main categories, each building on the previous one:

**Foundation LLMs (Base LLMs)** are general-purpose models trained on vast, diverse datasets encompassing text, code, images, and other modalities. Their extensive pretraining enables them to support various tasks across domains: NLP, computer vision, and speech recognition. They are powerful and flexible but require further adaptation for specific task requirements. A foundation LLM simply predicts the next word based on learned linguistic patterns — if you type "The capital of France is," it will likely continue with "Paris" because that pattern appears frequently in training data.

*Limitation for testing:* If you give a foundation LLM a user story and ask it to generate test cases, it might continue the text in an unpredictable way — perhaps writing more requirements instead of test cases — because it has no instruction-following training.

**Instruction-Tuned LLMs** are derived from foundation models and then fine-tuned using datasets that pair prompts with expected responses. This fine-tuning enhances alignment with human instructions, making the model better at task adherence, instruction following, and response coherence. When you say "Generate test cases for this user story," an instruction-tuned model understands that you want test cases as output, not a continuation of the story.

*Example:* ChatGPT is an instruction-tuned version of GPT. When you ask it to "List five boundary value test cases for an age field that accepts 18-65," it understands the task format and delivers a numbered list of test cases.

**Reasoning LLMs** extend instruction-tuned models by emphasizing structured cognitive abilities: logical inference, multi-step problem-solving, and chain-of-thought reasoning. They are further trained on carefully selected tasks demanding contextual understanding, intermediate reasoning steps, and complex information synthesis.

*Example:* Given a complex requirement with multiple interacting conditions (e.g., "Discount applies if the customer is a Gold member AND order total exceeds $100, unless the item is already on sale AND the sale discount is greater than the loyalty discount"), a reasoning LLM would systematically work through each condition combination, identify boundary cases, and generate a comprehensive decision table — showing its step-by-step thinking.

**Comparison Table**

| Aspect | Foundation LLM | Instruction-Tuned LLM | Reasoning LLM |
|--------|---------------|----------------------|---------------|
| Training | Pre-trained on vast diverse data | Fine-tuned with prompt-response pairs | Further trained on reasoning tasks |
| Primary Strength | Broad knowledge base | Following instructions accurately | Multi-step logical reasoning |
| Behavior | Predicts next word | Follows user instructions | Shows step-by-step thinking |
| Testing Use Case | Limited direct use | Routine test tasks (generation, analysis) | Complex test planning, prioritization |
| Example Models | GPT-4 Base, LLaMA Base | ChatGPT, Claude, Gemini | o1, o3, Claude with extended thinking |
| Best For | Research, embedding | Test case generation, analysis, reporting | Complex logic testing, decision tables |

**When to choose which for testing:**
- Use **instruction-tuned** LLMs for straightforward tasks: generating test cases, writing test scripts, creating reports, analyzing requirements
- Use **reasoning** LLMs for complex tasks requiring logic: test planning with constraints, risk-based prioritization, analyzing complex business rules, debugging intricate test failures

---

#### 1.1.4 Multimodal LLMs and Vision-Language Models (K2)

**Learning Objective GenAI-1.1.4:** Summarize the basic principles of Multimodal LLMs and Vision-Language Models.

**Multimodal LLMs** extend the traditional transformer model to process multiple data modalities — text, images, sound, and video. They are trained on large, diverse datasets that enable them to learn relationships between different data types. Each data type is adapted for tokenization: images, for instance, are converted into embeddings using vision-language models before being processed by the transformer.

**Vision-Language Models** are a subset of multimodal LLMs that specifically integrate visual and textual information. They can perform tasks such as image captioning, visual question answering, and analyzing textual-visual consistency.

**Applications in Software Testing:**

1. **Screenshot Analysis:** Upload a screenshot of your application along with a user story, and ask the model to identify discrepancies between the expected UI and the actual UI. For example: "Here is the user story for our checkout page and a screenshot of the current implementation. Identify any inconsistencies."

2. **GUI Wireframe Testing:** Provide a wireframe image alongside textual requirements and ask the model to generate acceptance criteria that cover both visual and functional aspects.

3. **Visual Regression Detection:** Feed the model a baseline screenshot and a current screenshot, asking it to identify visual differences that might indicate regressions.

4. **Accessibility Testing:** Upload a screenshot and ask the model to evaluate color contrast, font sizes, and layout against accessibility standards (WCAG).

*Example Prompt for Multimodal Testing:*
```
Role: You are a senior QA engineer specializing in UI testing.
Context: We are testing an e-commerce checkout page.
Instruction: Analyze the attached screenshot against the acceptance criteria below 
and identify any discrepancies.
Input Data: [Screenshot attached] 
Acceptance Criteria:
- The "Place Order" button should be green (#28a745)
- The order summary should show item count, subtotal, tax, and total
- The shipping address should be displayed above the payment section
Constraints: Focus only on visual and layout issues, not functionality.
Output Format: A numbered list of discrepancies with severity (High/Medium/Low).
```

**Hands-On Objective HO-1.1.4 (H1): Review and execute prompt addressing test task using multimodal LLM**

*Exercise:*
1. Take a screenshot of any web application you use
2. Write a user story that describes the expected behavior of that page
3. Create a prompt that includes both the screenshot and the user story
4. Submit to a multimodal LLM (GPT-4 Vision, Claude, Gemini)
5. Evaluate: Did the model correctly identify the relationship between the visual elements and the textual requirements? What did it miss?

---

### 1.2 Leveraging Generative AI in Software Testing: Core Principles

#### 1.2.1 Key LLM Capabilities for Test Tasks (K2)

**Learning Objective GenAI-1.2.1:** Give examples of key LLM capabilities for test tasks.

LLMs can interpret requirements, specifications, screenshots, code, test cases, and defect reports. Here are the key capabilities with concrete examples:

**1. Requirements Analysis and Improvement**

LLMs can analyze requirements and test basis elements to identify ambiguities, inconsistencies, and missing information. They can also generate meaningful questions for stakeholder clarification.

*Example Prompt:*
```
Analyze the following user story for testability issues:

"As a user, I want to be able to search for products so that I can find 
what I need quickly."

Identify: ambiguities, missing acceptance criteria, untestable elements, 
and suggest improvements.
```

*Expected Output:* The LLM might identify that "quickly" is ambiguous (needs a specific response time), that there are no acceptance criteria for empty results, partial matches, or search filters, and that "products" needs clarification on scope.

**2. Test Case Creation Support**

LLMs can generate test cases from requirements, user stories, and other test basis elements.

*Example Prompt:*
```
Generate functional test cases for:
User Story: As a registered user, I want to reset my password via email 
so that I can regain access to my account.
Acceptance Criteria:
- User clicks "Forgot Password" link on login page
- System sends reset email to registered email address
- Reset link expires after 24 hours
- New password must meet complexity requirements (8+ chars, 1 uppercase, 1 number)

Format: Test ID | Precondition | Steps | Expected Result | Priority
```

**3. Test Oracle Generation**

LLMs can help generate expected results for test cases, which is particularly useful when requirements are complex and calculating expected outcomes manually is time-consuming.

*Example:* Given a complex pricing formula with multiple discount tiers, an LLM can calculate expected prices for various test scenarios and generate a truth table of expected outcomes.

**4. Test Data Generation**

LLMs can create synthetic test data that is representative, diverse, and privacy-preserving.

*Example Prompt:*
```
Generate 10 test data records for a customer registration form:
Fields: First Name, Last Name, Email, Phone, Date of Birth, Country
Include: 3 valid records, 3 with boundary values, 2 with invalid data, 
2 with special characters.
Format: CSV
Ensure no real personal data is used.
```

**5. Test Automation Support**

LLMs can generate test scripts from test case descriptions and improve existing scripts.

*Example Prompt:*
```
Convert the following manual test case into a Selenium WebDriver test script in Python:
Test Case: Verify successful login
1. Navigate to https://example.com/login
2. Enter username "testuser@example.com"
3. Enter password "TestPass123!"
4. Click the Login button
5. Verify the dashboard page is displayed with welcome message
Use Page Object Model pattern. Include explicit waits.
```

**6. Test Result Analysis**

LLMs can summarize test results, classify anomalies, and identify patterns.

*Example:* Feed a test execution log with 500 test results to an LLM and ask it to identify clusters of failures, potential root causes, and which failures might share a common defect.

**7. Testware Creation**

LLMs can create test plans, test reports, defect reports, and other documentation, and keep them updated as the project evolves.

*Example:* Provide project context and ask the LLM to draft a test plan following IEEE 829 format, or generate a sprint test summary report from raw execution data.

---

#### 1.2.2 Interaction Models for GenAI in Testing (K2)

**Learning Objective GenAI-1.2.2:** Compare interaction models when using GenAI for software testing.

There are two complementary approaches to using GenAI in testing:

**AI Chatbots** provide a user-friendly, conversational interface for direct communication with LLMs. Testers input questions, commands, or prompts and receive immediate, contextually aware responses. Techniques like prompt chaining allow iterative refinement of outputs.

*Best for:* Fast feedback, exploratory testing, onboarding new testers, ad-hoc analysis, clarifying test concepts, brainstorming test scenarios, non-technical stakeholder collaboration.

*Example workflow:* A tester copies a user story into ChatGPT, asks for test conditions, reviews them, refines the prompt to focus on edge cases, then asks for test data — all in a single conversational session.

**LLM-Powered Testing Applications** integrate LLM capabilities via APIs into testing tools and frameworks. They perform well-defined, often automated testing tasks and offer greater customization and scalability.

*Best for:* Automated pipelines, CI/CD integration, large-scale test generation, consistent and repeatable processes, enterprise environments.

*Example workflow:* A test automation framework calls an LLM API to generate test data before each test run, analyze test results after execution, and generate defect reports for failures — all without human intervention.

**Comparison Table**

| Dimension | AI Chatbot | LLM-Powered Testing Application |
|-----------|-----------|-------------------------------|
| Interface | Conversational, browser-based | API-based, integrated into tools |
| User Skill | Low barrier to entry | Requires development skills |
| Customization | Limited (prompt-based) | Highly customizable |
| Scalability | Manual, one-at-a-time | Automated, batch processing |
| Consistency | Varies with each interaction | Reproducible (same API calls) |
| Integration | Standalone | Embedded in test frameworks |
| Cost Model | Subscription or per-message | API usage (per-token pricing) |
| Best Use | Exploratory, ad-hoc, creative | Systematic, repetitive, automated |

**Critical Success Factor:** Regardless of the interaction model, strong prompt engineering is required. A poorly written prompt produces poor results whether typed into a chatbot or sent via API. This is why Chapter 2 dedicates 365 minutes to prompt engineering.

---

## Chapter 2: Prompt Engineering for Effective Software Testing

**Time Allocation:** 365 minutes (~6 hours)

### Keywords to Remember

**Standard Testing Keywords:** acceptance criteria, test script, test case, test condition, test data, test design, test report

**GenAI-Specific Keywords:**

| Term | Quick Definition |
|------|-----------------|
| Few-shot prompting | Providing multiple examples in the prompt to guide the LLM |
| Meta prompting | Using AI to generate or refine its own prompts |
| Natural language processing (NLP) | Computer processing of human language data |
| One-shot prompting | Providing one example in the prompt |
| Prompt | Natural language input provided to elicit a specific response |
| Prompt chaining | Using one prompt's output as the next prompt's input |
| Prompt engineering | Designing and refining prompts to guide LLMs toward desired outputs |
| System prompt | Hidden instruction set establishing context and rules for the LLM |
| User prompt | The actual input/question from the chatbot user |
| Zero-shot prompting | Providing no examples, relying on the model's pre-existing knowledge |

---

### 2.1 Effective Prompt Development

#### 2.1.1 Structure of Prompts for Software Testing (K2)

**Learning Objective GenAI-2.1.1:** Give examples of the structure of prompts used in generative AI for software testing.

A well-structured prompt has six key components. Think of them as a recipe: leaving out ingredients leads to unpredictable results.

**1. Role** — Who should the AI pretend to be?

The role defines the perspective and persona for the GenAI model. It determines the tone, depth, and approach of the response.

*Examples:*
- "You are a senior test analyst with 10 years of experience in financial software testing."
- "You are a test automation engineer specializing in Selenium and API testing."
- "You are a security testing expert familiar with OWASP Top 10."

*Why it matters:* Telling the AI it is a "senior test analyst" produces more thorough, professionally worded test cases than simply asking it to "write test cases." The role primes the model to draw on relevant patterns from its training.

**2. Context** — What is the background?

Context provides details about the test object, specific functionality, domain, and relevant situational information.

*Examples:*
- "We are testing a mobile banking application for iOS and Android. The app allows users to transfer money between accounts."
- "This is a healthcare system that must comply with HIPAA regulations. Patient data privacy is paramount."

**3. Instruction** — What exactly should the AI do?

Instructions must be clear, imperative, and concise. They outline the specific task to be performed.

*Examples:*
- "Generate negative test cases that attempt to break the input validation."
- "Analyze the following requirements and identify any ambiguities or contradictions."
- "Create a test execution schedule that prioritizes high-risk test cases first."

**4. Input Data** — What information does the AI need?

This includes user stories, acceptance criteria, screenshots, code, existing test cases, or any data the AI needs to perform the task.

*Examples:*
- A complete user story with acceptance criteria
- Source code for a function to be tested
- A test report from the previous sprint
- A screenshot of the application's current state

**5. Constraints** — What are the boundaries?

Constraints specify restrictions, limitations, or special considerations.

*Examples:*
- "Only generate test cases for the payment flow, not the registration flow."
- "Test data must not contain any real personal information."
- "Focus on boundary values and equivalence partitions only."
- "Do not exceed 10 test cases."

**6. Output Format** — How should the response be structured?

Specifying the output format ensures results meet your specific needs and are immediately usable.

*Examples:*
- "Present results in a table with columns: Test ID, Description, Steps, Expected Result, Priority"
- "Use Gherkin format (Given/When/Then)"
- "Output as JSON that can be imported into our test management tool"
- "Write the test script in Python using pytest framework"

**Complete Worked Example — All 6 Components:**

```
[Role]
You are a senior QA engineer with expertise in e-commerce applications 
and payment processing.

[Context]
We are testing the checkout flow of an online retail application. 
The application supports credit card, debit card, and PayPal payments. 
The checkout process includes: cart review, shipping address, payment 
method selection, and order confirmation. We are in Sprint 12 and need 
to ensure payment processing works correctly before the holiday sale.

[Instruction]
Generate comprehensive test cases for the payment method selection 
and payment processing steps of the checkout flow.

[Input Data]
User Story: As a customer, I want to pay for my order using my 
preferred payment method so that I can complete my purchase.

Acceptance Criteria:
- AC1: Customer can select Credit Card, Debit Card, or PayPal
- AC2: Credit/Debit card numbers are validated using Luhn algorithm
- AC3: Expired cards are rejected with clear error message
- AC4: PayPal redirects to PayPal login page
- AC5: Successful payment shows order confirmation with order number
- AC6: Failed payment shows retry option without losing cart contents

[Constraints]
- Include both positive and negative test cases
- Cover boundary values for card number and expiry date fields
- Do not include test cases for the cart review or shipping address steps
- Assume the user is already logged in

[Output Format]
Table format with columns:
| TC-ID | Category | Description | Preconditions | Test Steps | Expected Result | Priority (H/M/L) |
```

---

#### 2.1.2 Core Prompting Techniques (K2)

**Learning Objective GenAI-2.1.2:** Differentiate core prompting techniques for software testing.

Three core techniques enhance basic prompts:

**1. Prompt Chaining**

Prompt chaining breaks a task into a series of intermediate steps, where the result of each step is checked (manually or automatically) and refined before feeding into the next prompt.

*Why it works:* Complex tasks are more error-prone when handled in a single prompt. By breaking them into subtasks, you can verify each intermediate output and correct errors before they propagate.

*Step-by-Step Testing Example:*

**Step 1 — Analyze requirements:**
```
Analyze the following user story and identify all testable conditions:

"As an admin, I want to set user permissions by role so that users 
only access features appropriate to their level."

Roles: Viewer (read-only), Editor (read/write), Admin (full access)
Features: Dashboard, Reports, User Management, Settings
```
*Review the output. Correct any missing conditions.*

**Step 2 — Generate test cases from conditions:**
```
Using the following test conditions (from previous analysis), generate 
detailed test cases:

[Paste verified test conditions from Step 1]

For each test condition, create at least one positive and one negative test case.
```
*Review the output. Check completeness and accuracy.*

**Step 3 — Create test data:**
```
For the following test cases, generate the necessary test data:

[Paste verified test cases from Step 2]

Include user accounts for each role with appropriate permission sets.
```

Each step builds on verified output from the previous step, leading to higher quality overall.

**2. Few-Shot Prompting**

Few-shot prompting provides the LLM with examples in the prompt to guide its output. There are three variants:

**Zero-Shot Prompting** (no examples):
```
Generate test cases for a password reset feature.
```
The model relies entirely on its pre-existing knowledge. Results are often generic and may not match your format or style.

**One-Shot Prompting** (one example):
```
Generate test cases for a password reset feature. Follow this format:

Example:
TC-001 | Valid Reset | Enter registered email, click reset | 
User receives reset email within 2 minutes | High

Now generate similar test cases for all scenarios.
```
One example dramatically improves format consistency.

**Few-Shot Prompting** (multiple examples):
```
Generate test cases for a password reset feature using these examples as a guide:

Example 1 (Positive):
TC-001 | Valid Reset Request | Precondition: User has registered account | 
Steps: 1. Click "Forgot Password" 2. Enter registered email 3. Click Submit | 
Expected: Reset email sent within 2 minutes | Priority: High

Example 2 (Negative):
TC-002 | Unregistered Email | Precondition: Email not in system | 
Steps: 1. Click "Forgot Password" 2. Enter unregistered email 3. Click Submit | 
Expected: Generic message "If this email exists, you'll receive a reset link" 
(no information leakage) | Priority: High

Example 3 (Boundary):
TC-003 | Expired Reset Link | Precondition: Reset link older than 24 hours | 
Steps: 1. Click expired reset link | 
Expected: "This link has expired. Please request a new one." | Priority: Medium

Now generate additional test cases covering remaining scenarios.
```
Multiple examples consolidate the desired pattern, ensuring consistent, high-quality output.

**3. Meta Prompting**

Meta prompting leverages the AI's ability to generate and refine its own prompts. Instead of writing the perfect prompt yourself, you collaborate with the LLM to create it.

*Example:*
```
I need to test a flight booking system. The system handles search, 
seat selection, payment, and booking confirmation. 

Help me create an effective prompt that I can use to generate 
comprehensive test cases for the seat selection feature. 
The prompt should follow best practices for prompt engineering 
and include role, context, instruction, input data, constraints, 
and output format.
```

The LLM generates a prompt template. You review it, adjust it, and then use the refined prompt for the actual test case generation. This is particularly powerful when you are unsure how to craft an effective prompt — it is like "pair testing" with the AI.

**Comparison Table: Prompting Techniques**

| Aspect | Prompt Chaining | Few-Shot Prompting | Meta Prompting |
|--------|----------------|-------------------|----------------|
| Approach | Break task into sequential steps | Provide examples in prompt | AI generates/refines prompts |
| Best For | Complex, multi-step tasks | Repetitive tasks with specific format | New or unfamiliar tasks |
| Human Effort | Verify each intermediate step | Prepare good examples upfront | Review and refine AI-generated prompt |
| Key Benefit | Precision through verification | Consistency through examples | Efficiency in prompt creation |
| Risk | Time-consuming for simple tasks | Bad examples lead to bad output | Generated prompt may need significant editing |
| Testing Examples | Requirements → Conditions → Test Cases | Gherkin-style generation, keyword-driven tests | Creating prompts for unfamiliar domains |

---

#### 2.1.3 System Prompts vs User Prompts (K2)

**Learning Objective GenAI-2.1.3:** Distinguish between system prompts and user prompts.

**System Prompt** — the hidden instructions:

A system prompt is typically defined by a developer or tester and guides the overall LLM behavior. It is not visible or editable by the chatbot user in most interfaces. It acts as a predefined command set that defines the LLM's behavior, personality, and operational parameters.

*Example System Prompt for a Testing Assistant:*
```
You are a professional software testing assistant specializing in 
ISTQB-aligned practices. 

Rules:
- Always respond clearly using formal, professional language
- Focus on testing best practices and cite ISTQB principles when relevant
- When generating test cases, always include preconditions and expected results
- Avoid speculation — if you are unsure, say so
- When asked about test techniques, reference the appropriate ISTQB 
  test design technique by name
- Never generate test data containing real personal information
- Always flag potential risks and suggest mitigations
```

The system prompt stays constant throughout the interaction session. It establishes the fundamental framework for all LLM responses.

**User Prompt** — the actual question:

The user prompt is the input or question from the chatbot user. It changes with each interaction and can include specific instructions, questions, and tasks.

*Example User Prompt:*
```
List the key differences between black-box and white-box testing 
with examples relevant to API testing.
```

**Typical Usage Pattern:**
1. Set the system prompt once at conversation start
2. Send successive user prompts for each interaction
3. The LLM generates responses considering both the unchanging system prompt AND the current user prompt

This separation is powerful for testing tools: the system prompt ensures consistent behavior (always following ISTQB practices, always including expected results), while user prompts handle the specific task at hand.

---

### 2.2 Applying Prompt Engineering Techniques to Software Test Tasks

#### 2.2.1 Test Analysis Tasks (K3)

**Learning Objective GenAI-2.2.1:** Apply generative AI to test analysis tasks.

GenAI supports test analysis by generating and prioritizing test conditions, identifying test basis defects, and providing coverage analysis. The quality and relevance of inputs directly impact the accuracy of LLM output.

**Typical Test Analysis Tasks:**

**1. Identifying Potential Defects in Test Basis**

*Complete Worked Example:*

```
Role: You are an experienced test analyst reviewing requirements for testability.

Context: We are developing a hotel booking system. The following requirement 
was written by a junior business analyst.

Instruction: Analyze this requirement and identify all ambiguities, 
inconsistencies, missing information, and potential defects.

Input Data:
REQ-101: "The system should allow users to book rooms quickly. 
The booking form should validate dates and show available rooms. 
Payment should be processed securely. Users can cancel bookings 
according to the cancellation policy."

Constraints: Focus on testability issues. For each issue found, 
suggest a specific improvement.

Output Format:
| Issue # | Type (Ambiguity/Inconsistency/Missing/Defect) | Description | 
Suggested Improvement | Severity |
```

*Expected findings:* "Quickly" is ambiguous (needs response time), "validate dates" is vague (what validation?), "securely" needs specific security standards, "cancellation policy" is not defined, no mention of error handling, no accessibility requirements.

**2. Generating Test Conditions from Test Basis**

The LLM analyzes requirements or user stories and breaks them down into specific, measurable, testable conditions.

**3. Prioritizing Test Conditions by Risk**

Provide the LLM with risk likelihood and risk impact for each test condition, and it helps prioritize test effort based on regulatory compliance, user-facing features, and historical defect data.

**4. Supporting Coverage Analysis**

The LLM maps requirements to test conditions and identifies gaps where aspects of the test basis are not yet covered.

**5. Suggesting Test Techniques**

Based on the type of requirement, the LLM recommends relevant test techniques such as boundary value analysis, equivalence partitioning, decision tables, or state transition testing.

---

#### 2.2.2 Test Design and Implementation Tasks (K3)

**Learning Objective GenAI-2.2.2:** Apply generative AI to test design and test implementation tasks.

**Complete Worked Example — Gherkin Test Case Generation Using Few-Shot Prompting:**

```
Role: You are a BDD test specialist.

Context: We are writing Gherkin scenarios for a library management system.

Instruction: Generate Gherkin test scenarios for the user story below. 
Follow the style of the examples provided.

Examples:

User Story: As a member, I want to search for books by title.
Test Conditions: Search with exact title, partial title, non-existent title.

Scenario: Search with exact book title
  Given the library has the book "Clean Code"
  When I search for "Clean Code"
  Then I should see "Clean Code" in the search results
  And the result should show author "Robert C. Martin"

Scenario: Search with partial book title
  Given the library has the book "Clean Code"
  When I search for "Clean"
  Then I should see "Clean Code" in the search results

Scenario: Search with non-existent title
  Given the library has no book titled "Nonexistent Book"
  When I search for "Nonexistent Book"
  Then I should see a message "No books found matching your search"

Input Data:
User Story: As a member, I want to borrow a book so that I can read it at home.
Acceptance Criteria:
- AC1: Member can borrow up to 5 books at a time
- AC2: Borrowed book's status changes to "Checked Out"
- AC3: Due date is set to 14 days from borrowing date
- AC4: Member cannot borrow a book that is already checked out by another member

Constraints: Cover all acceptance criteria. Include at least one negative scenario 
for each AC.

Output Format: Gherkin (Given/When/Then)
```

**Test Data Synthesis:**

GenAI can create representative, privacy-preserving synthetic test data that covers extreme situations and varied conditions without exposing sensitive information.

**Automated Test Script Generation:**

GenAI can translate structured test cases into code compatible with various test automation frameworks, and update or extend scripts based on new requirements.

**Test Execution Scheduling and Prioritization:**

GenAI can analyze test cases and their interdependencies to optimize test execution schedules based on priority, associated risks, resource availability, and test objectives.

---

#### 2.2.3 Automated Regression Testing (K3)

**Learning Objective GenAI-2.2.3:** Apply generative AI to automated regression testing.

As iterations and releases accumulate, regression test suites grow, making them ideal candidates for automation — especially in CI/CD pipelines. GenAI streamlines this by assisting in creation, maintenance, and optimization of automated regression tests.

**Key Activities GenAI Supports:**

**1. Keyword-Driven Test Script Implementation**

LLMs can map pre-defined keywords to specific test cases and generate test scripts using keyword-driven frameworks.

*Example:* Given keywords like `OpenBrowser`, `NavigateTo`, `EnterText`, `ClickButton`, `VerifyText`, the LLM generates a complete test script:

```
Role: You are a test automation engineer working with a keyword-driven framework.

Instruction: Given the following keyword library and test case, generate 
the keyword-driven test script.

Keyword Library:
- OpenBrowser(browser_type)
- NavigateTo(url)
- EnterText(locator, text)
- ClickButton(locator)
- VerifyText(locator, expected_text)
- VerifyElementVisible(locator)
- WaitForElement(locator, timeout_seconds)
- CloseBrowser()

Test Case: Verify successful login with valid credentials
Precondition: User "testuser" exists with password "Pass123!"
```

**2. Impact Analysis and Test Optimization**

GenAI analyzes code changes to identify high-risk areas, enabling targeted regression testing where it is most needed. By reviewing a pull request diff, the LLM can identify which test suites are most relevant to the changed code.

**3. Self-Healing and Adaptive Tests**

GenAI can automatically adjust test scripts to handle minor UI or API changes — such as changed element locators or modified response schemas — preventing unnecessary test failures from small modifications.

**4. Automated Test Reporting and Insights**

GenAI generates detailed test reports with success metrics, failures, key insights, and predictive insights on potential failure points.

**5. Enhanced Defect Reporting and Root Cause Analysis**

GenAI supports automatic compilation of comprehensive defect reports including test logs, screenshots, and test environment data.

**Important Caveat:** GenAI can make mistakes. Generated output must be carefully checked depending on the associated risk (see Chapter 3).

---

#### 2.2.4 Test Control and Monitoring Tasks (K3)

**Learning Objective GenAI-2.2.4:** Apply generative AI to test control and monitoring tasks.

Test monitoring requires retrieving large quantities of often unstructured data. GenAI can analyze and synthesize this data effectively.

**1. Test Monitoring and Metrics Analysis:** Automate trend analysis, predict potential risks, alert teams of deviations from plan.

**2. Test Control:** Provide insights for reprioritizing tests, adjusting schedules, and reallocating resources based on current status.

**3. Test Completion Insights:** Generate completion reports highlighting successes, lessons learned, and recommendations for future improvement.

**4. Enhanced Visualization and Reporting:** Create dynamic dashboards and natural language summaries that make test metrics accessible to all stakeholders.

---

#### 2.2.5 Selecting Appropriate Prompting Techniques (K3)

**Learning Objective GenAI-2.2.5:** Select and apply appropriate prompting techniques for a given context and test task.

**Decision Guide:**

| Technique | When to Use | Key Features | Example Test Task |
|-----------|------------|-------------|-------------------|
| Prompt Chaining | Complex tasks needing precision with human verification at each step | Breaks into subtasks; each step verified | Requirements analysis → test conditions → test cases |
| Few-Shot Prompting | Repetitive tasks or tasks requiring specific output format | Examples guide consistent output | Gherkin scenarios, keyword-driven scripts, formatted reports |
| Meta Prompting | New/unfamiliar tasks; need help crafting the right prompt | AI helps create the prompt itself | Creating prompts for a new testing domain |

**Key Insight:** Multiple techniques can be combined for a single use case:
1. Start with **meta prompting** to create the initial prompt
2. Enhance with **few-shot** examples for format consistency
3. Use **prompt chaining** to break complex work into verified steps

---

### 2.3 Evaluate Generative AI Results and Refine Prompts

#### 2.3.1 Metrics for Evaluating GenAI Results (K2)

**Learning Objective GenAI-2.3.1:** Understand the metrics for evaluating the results of generative AI on test tasks.

| Metric | Description | Testing Example |
|--------|-------------|-----------------|
| **Accuracy** | Overall correctness of output against expert-written references | Do generated test cases cover all specified requirements? |
| **Precision** | Correctness relative to a specific objective | Do generated test cases correctly identify real anomalies (not false positives)? |
| **Recall** | Ability to identify all relevant instances | Do generated test cases cover ALL valid and invalid equivalence partitions? |
| **Relevance & Contextual Fit** | Whether output is applicable and appropriate for the given context | Are test cases consistent with the test basis and domain requirements? |
| **Diversity** | Range of inputs and scenarios covered, avoiding repetition | Do test cases cover various user behaviors and edge cases? |
| **Execution Success Rate** | Proportion of generated test scripts that execute successfully | How many scripts run without syntax errors or format issues? |
| **Time Efficiency** | Time saved compared to manual effort | AI generates 50 test cases in 2 minutes vs. 4 hours manually |

**Evaluation Methods:**
- **Manual reviews** by experienced testers
- **Automated evaluation** comparing LLM output against predefined reference sets
- **Critical requirement:** Metrics must be based on statistically relevant data, given GenAI's non-deterministic nature. Running a prompt once and measuring is insufficient.

---

#### 2.3.2 Techniques for Evaluating and Refining Prompts (K2)

**Learning Objective GenAI-2.3.2:** Give examples of techniques for evaluating and iteratively refining prompts.

**1. Iterative Prompt Modification:** Start with a base prompt, evaluate results, and iteratively add context, adjust wording, and improve specificity.

**2. A/B Testing of Prompts:** Create multiple prompt versions and evaluate which produces better results. For example, test whether including a role improves test case quality compared to a prompt without a role.

**3. Output Analysis:** Examine AI-generated output for inaccuracies, inconsistencies, or patterns of errors. Understanding error types helps refine prompts to avoid similar issues.

**4. User Feedback Integration:** Gather input from testers on the usefulness and clarity of generated output. Use insights to refine prompts for real-world needs.

**5. Adjust Prompt Length and Specificity:** Experiment with different levels of detail. Sometimes more context improves quality; other times, shorter prompts yield better generalization.

**Organizational Best Practice:** Test teams should organize prompt evaluation and optimization sessions, share prompt libraries across projects and domains, and build a culture of continuous improvement.

---

## Practice Exercises — Chapters 1 and 2

**Instructions:** Select the BEST answer for each question.

**Q1 (K1).** Which type of AI uses symbols, rules, and structured knowledge to model reasoning?
A) Deep Learning
B) Classical Machine Learning
C) Symbolic AI
D) Generative AI

**Q2 (K2).** An LLM generates a test case that references an acceptance criterion not present in the user story. This is an example of:
A) A reasoning error
B) A hallucination
C) Bias
D) Non-deterministic behavior

**Q3 (K2).** What is the primary difference between a foundation LLM and an instruction-tuned LLM?
A) Foundation LLMs are larger
B) Instruction-tuned LLMs are fine-tuned to follow user instructions
C) Foundation LLMs cannot generate text
D) Instruction-tuned LLMs only work with code

**Q4 (K1).** What is tokenization?
A) Encrypting text for security
B) Breaking text into smaller units for LLM processing
C) Converting text to binary
D) Compressing text to fit context window

**Q5 (K2).** A tester is generating test cases for a complex insurance claim workflow. Which prompting technique is MOST appropriate?
A) Zero-shot prompting
B) Few-shot prompting
C) Prompt chaining
D) Meta prompting

**Q6 (K2).** Which component of a structured prompt defines the perspective and persona for the GenAI model?
A) Context
B) Role
C) Instruction
D) Constraints

**Q7 (K3).** A tester needs to generate Gherkin-style test cases that follow a specific format used by their organization. Which prompting technique is MOST effective?
A) Zero-shot prompting
B) Meta prompting
C) Few-shot prompting
D) Prompt chaining

**Q8 (K2).** What is the key difference between a system prompt and a user prompt?
A) System prompts are longer
B) System prompts are visible to the user; user prompts are hidden
C) System prompts set overall behavior and remain constant; user prompts change per interaction
D) User prompts cannot contain instructions

**Q9 (K2).** Which metric measures the proportion of generated test cases that can be executed without syntax errors?
A) Accuracy
B) Precision
C) Execution Success Rate
D) Recall

**Q10 (K2).** A vision-language model would be MOST useful for which testing task?
A) Generating SQL test data
B) Comparing a screenshot against UI acceptance criteria
C) Calculating test effort estimates
D) Writing a test plan

**Q11 (K2).** What is a context window?
A) The number of characters in a prompt
B) The span of text (in tokens) a model considers when generating responses
C) The time limit for model responses
D) The maximum number of prompts per session

**Q12 (K3).** A tester is unfamiliar with prompt engineering and wants to create effective prompts for a new testing domain. Which technique should they start with?
A) Zero-shot prompting
B) Few-shot prompting
C) Meta prompting
D) Prompt chaining

**Q13 (K2).** Which of the following is an advantage of LLM-powered testing applications over AI chatbots?
A) Lower cost
B) Easier to use for non-technical stakeholders
C) Greater customization and scalability
D) No prompt engineering needed

**Q14 (K2).** When evaluating GenAI results, why must metrics be based on statistically relevant data?
A) Because GenAI is expensive to run
B) Because of GenAI's non-deterministic behavior
C) Because regulators require it
D) Because GenAI only works with large datasets

**Q15 (K3).** A tester wants to use prompt chaining for requirements analysis. What is the correct approach?
A) Send all requirements in one prompt and ask for test cases
B) Break the task into steps: analyze requirements, then verify conditions, then generate test cases — checking each step
C) Provide examples of good requirements analysis
D) Ask the LLM to create its own prompt for requirements analysis

**Answer Key:**
Q1: C | Q2: B | Q3: B | Q4: B | Q5: C | Q6: B | Q7: C | Q8: C | Q9: C | Q10: B | Q11: B | Q12: C | Q13: C | Q14: B | Q15: B

---

## Additional Topics: Modern AI Developments

### Agentic AI and Autonomous Testing Agents

Beyond simple chatbot interactions, the industry is moving toward agentic AI — autonomous agents that can plan, reason, and execute multi-step testing tasks with minimal human intervention. These agents use LLMs as their reasoning engine but augment them with tools (web browsers, code interpreters, APIs) to interact with systems under test. For example, an agentic testing system might autonomously explore a web application, identify features, generate test cases, execute them, and report defects.

### AI Coding Assistants for Test Automation

Tools like GitHub Copilot, Cursor, and Amazon CodeWhisperer are transforming how testers write automation code. They provide real-time code suggestions, complete test methods, and generate assertion patterns based on context. While these are not purely GenAI testing tools, they significantly accelerate test script development and are rapidly becoming part of the tester's toolkit.

### RAG for Testing (Preview for Chapter 4)

Retrieval-Augmented Generation (RAG) combines LLMs with company-specific data sources, enabling test tools to access the latest requirements, specifications, and existing test data when generating new test artifacts. This addresses one of the biggest limitations of GenAI — being out of date with your project's current state.

### AI Safety and Alignment Considerations

As AI becomes more integrated into testing, testers need to understand alignment — ensuring AI systems behave as intended. This includes understanding guardrails, safety filters, and the emerging field of AI safety testing, which evaluates whether AI systems operate within acceptable boundaries even under adversarial conditions.

---

## Index of Key Terms

| Term | Section |
|------|---------|
| A/B testing of prompts | 2.3.2 |
| Acceptance criteria | 2.2.1, 2.2.2 |
| Accuracy (metric) | 2.3.1 |
| AI chatbot | 1.2.2 |
| Agentic AI | Additional Topics |
| Classical Machine Learning | 1.1.1 |
| Context (prompt component) | 2.1.1 |
| Context window | 1.1.2 |
| Constraints (prompt component) | 2.1.1 |
| Deep learning | 1.1.1 |
| Diversity (metric) | 2.3.1 |
| Embedding | 1.1.2 |
| Execution Success Rate | 2.3.1 |
| Feature | Keywords |
| Few-shot prompting | 2.1.2 |
| Foundation LLM | 1.1.3 |
| Generative AI | 1.1.1, 1.1.2 |
| Generative pre-trained transformer | Keywords |
| Gherkin | 2.2.2 |
| Input Data (prompt component) | 2.1.1 |
| Instruction (prompt component) | 2.1.1 |
| Instruction-tuned LLM | 1.1.3 |
| Iterative prompt modification | 2.3.2 |
| Keyword-driven test automation | 2.2.3 |
| Large language model (LLM) | 1.1.2 |
| LLM-powered testing applications | 1.2.2 |
| Machine learning | 1.1.1 |
| Meta prompting | 2.1.2 |
| Multimodal model | 1.1.4 |
| Natural language processing (NLP) | Keywords |
| Non-deterministic behavior | 1.1.2 |
| One-shot prompting | 2.1.2 |
| Output Analysis | 2.3.2 |
| Output Format (prompt component) | 2.1.1 |
| Precision (metric) | 2.3.1 |
| Prompt | 2.1.1 |
| Prompt chaining | 2.1.2 |
| Prompt engineering | 2.1 |
| Reasoning LLM | 1.1.3 |
| Recall (metric) | 2.3.1 |
| Relevance and Contextual Fit | 2.3.1 |
| Role (prompt component) | 2.1.1 |
| Self-healing tests | 2.2.3 |
| Small Language Model (SLM) | 1.1.2 |
| Symbolic AI | 1.1.1 |
| System prompt | 2.1.3 |
| Test analysis | 2.2.1 |
| Test case generation | 2.2.2 |
| Test condition | 2.2.1 |
| Test data synthesis | 2.2.2 |
| Test monitoring | 2.2.4 |
| Test oracle generation | 1.2.1 |
| Test report | 2.2.3 |
| Test script generation | 2.2.2 |
| Time Efficiency (metric) | 2.3.1 |
| Tokenization | 1.1.2 |
| Transformer | 1.1.2 |
| User prompt | 2.1.3 |
| Vision-language model | 1.1.4 |
| Zero-shot prompting | 2.1.2 |

---

## References

1. ISTQB CT-GenAI Syllabus v1.0 (2025)
2. ISTQB Foundation Level Syllabus v4.0 (2023)
3. ISO/IEC 42001:2023 — AI Management System
4. ISO/IEC 23053:2022 — Framework for AI Systems Using ML
5. EU AI Act — Regulation (EU) 2024/1689
6. NIST AI Risk Management Framework (AI RMF 1.0)
7. OpenAI Tokenizer — https://platform.openai.com/tokenizer
8. Vaswani, A. et al. (2017). "Attention Is All You Need" — Original transformer paper
9. Brown, T. et al. (2020). "Language Models are Few-Shot Learners" — GPT-3 paper
10. ISTQB Glossary — https://glossary.istqb.org

---

*End of Part 1 — Continue to Part 2 for Chapters 3-5: Risks, Infrastructure & Deployment*
