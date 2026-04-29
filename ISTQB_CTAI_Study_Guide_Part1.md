# ISTQB CT-AI Study Guide — Part 1: AI Fundamentals, Quality & Machine Learning

> **Based on:** ISTQB Certified Tester AI Testing (CT-AI) Syllabus v2.0
> **Chapters Covered:** 1, 2, and 3
> **Total Study Time:** 120 + 45 + 375 = 540 minutes (~9 hours)

---

## Exam Overview

The CT-AI certification requires CTFL as prerequisite. The exam tests at four cognitive levels: K1 (Remember), K2 (Understand), K3 (Apply), K4 (Analyze). Hands-on objectives: H0 (demo), H1 (guided), H2 (with hints). Total training: 19.5 hours minimum.

### Business Outcomes (BO1-BO8)
- **BO1:** Understand the current state of AI including generative AI
- **BO2:** Experience implementing and testing ML models
- **BO3:** Understand working and testing of simple neural networks
- **BO4:** Understand AI quality characteristics (ISO/IEC 25059)
- **BO5:** Calculate and interpret ML functional performance metrics
- **BO6:** Recognize scope of ML-specific test levels
- **BO7:** Contribute to effective test strategy for ML systems
- **BO8:** Design and execute test cases for ML systems

---

## Chapter 1: Introduction to Artificial Intelligence (120 minutes)

### Keywords
AI-based system, artificial intelligence, general AI, machine learning, ML development framework, narrow AI, super AI

---

### 1.1.1 AI-Based Systems vs Conventional Systems (K2)

**Learning Objective AI-1.1.1:** Differentiate between AI-based systems and conventional systems.

**Conventional Systems** are programmed using imperative languages with explicit if-then-else statements and loops. Their behavior is deterministic and predictable — given the same input, you always get the same output. Every decision can be traced to a specific line of code.

*Example:* A calculator app that computes tax. Input $1000 at 10% → always outputs $100. You can read the code and verify exactly how it works.

**AI-Based Systems** learn patterns from data without predefined rules. They use probabilistic reasoning and statistical inference, producing potentially unpredictable outputs. Many AI models (especially deep learning) are "black-box" due to billions of parameters, making decisions difficult to explain.

*Example:* A spam filter that learns from thousands of emails. It may classify a new email differently than a human would expect, and explaining why is challenging — the decision emerges from complex learned patterns.

**Comparison Table**

| Dimension | Conventional System | AI-Based System |
|-----------|-------------------|-----------------|
| Programming | Explicit rules (if-then-else) | Learns patterns from data |
| Behavior | Deterministic, predictable | Probabilistic, may vary |
| Explainability | Fully traceable to code | Often "black-box" |
| Adaptability | Requires reprogramming | Can self-learn and adapt |
| Testing | Known expected outputs | Statistical/probabilistic verification |
| Maintenance | Update rules manually | May need retraining, monitoring for drift |
| Data dependency | Operates on defined inputs | Highly dependent on training data quality |
| Error patterns | Systematic, reproducible bugs | Unpredictable, distribution-dependent errors |

**Testing Implications:** AI systems require fundamentally different testing approaches — statistical testing, monitoring for drift, testing for bias, and addressing the test oracle problem (determining correct output).

---

### 1.1.2 Narrow AI, General AI, and Super AI (K2)

**Learning Objective AI-1.1.2:** Distinguish between narrow AI, general AI, and super AI.

**Narrow AI (Weak AI):** Designed for specific tasks. ALL deployed AI systems today are narrow AI. Examples: image recognition, speech processing, language translation, spam filtering. Cannot generalize beyond learned functions.

**Frontier AI** is a subset of narrow AI representing the most advanced current systems (including GenAI models like GPT-4, Claude, Gemini).

**General AI (Strong AI):** Would possess the ability to perform most intellectual tasks a human can — understanding, learning, and applying knowledge across a broad range without retraining. Does NOT exist yet despite significant progress.

**Super AI (Artificial Superintelligence):** A theoretical AI that continuously improves itself without human intervention, surpassing human intelligence. The transition from general to super AI is called the **"technological singularity."** Considered an existential risk.

| Aspect | Narrow AI | General AI | Super AI |
|--------|----------|------------|---------|
| Status | Exists today | Does not exist | Theoretical |
| Scope | Specific tasks only | All intellectual tasks | Beyond human capability |
| Learning | Task-specific training | Cross-domain reasoning | Self-improving |
| Example | ChatGPT, image classifiers | Science fiction depictions | Hypothetical |
| Risk level | Manageable | Significant concerns | Existential risk |

---

### 1.1.3 Types of AI Technologies (K2)

**Learning Objective AI-1.1.3:** Explain different types of AI technologies.

**Machine Learning — Core Branch**

**Supervised Learning** uses labeled data (inputs paired with correct outputs):
- **Classification:** Assigning inputs to predefined categories. *Example:* Email → spam/not-spam; image → cat/dog.
- **ML Regression:** Predicting continuous numerical values. *Example:* Predicting house prices from features (size, location, age). **Important:** ML regression is different from regression testing in software testing.

**Unsupervised Learning** uses unlabeled data:
- **Clustering:** Grouping similar data points. *Example:* Customer segmentation for targeted marketing.
- **Association:** Identifying relationships between items. *Example:* "Customers who bought X also bought Y."

**Reinforcement Learning** — an agent learns through environment interaction, receiving rewards (positive) or penalties (negative):
- *Example:* A game-playing AI learns strategies through thousands of matches.
- *Applications:* Robotics, autonomous vehicles, adaptive chatbots.
- *Challenges:* Designing reward functions, setting up environments, strategy selection.

**Deep Learning (DL)** — Subset of ML using deep neural networks:
- **CNN (Convolutional Neural Networks):** Image recognition, object detection
- **RNN (Recurrent Neural Networks):** Sequential data — text, time series
- **Transformers:** Long-range dependencies, NLP, vision tasks

**Generative AI (GenAI):** Creates new content (text, images, audio, video) using LLMs that combine deep neural networks with NLP.

**Specialized Technologies:**
- **NLP:** Language analysis, sentiment analysis, machine translation
- **Computer Vision:** Visual data analysis, facial recognition
- **Fuzzy Logic:** Reasoning under uncertainty
- **Search Algorithms:** Optimization, navigation
- **Rule-Based/Expert Systems:** Structured decision support
- **Agentic AI:** Autonomous agents that plan, reason, and act

---

### 1.1.4 Generative AI (K2)

**Learning Objective AI-1.1.4:** Explain generative AI.

GenAI systems specialize in creating new content — text, images, video, music, data.

**Core Technologies:**
- **GANs (Generative Adversarial Networks):** Two neural networks compete — generator creates content, discriminator evaluates it
- **Diffusion Models:** Generate content by iteratively adding/removing noise
- **Transformers:** Self-attention mechanisms for coherent, contextual generation
- **Foundation Models:** Large-scale pretrained models fine-tuned for specific tasks

**Societal Concerns:** Deepfakes and misinformation, privacy/security/manipulation risks, employment impact, sustainability (high energy consumption), regulatory responses (EU AI Act).

---

### 1.1.5 Hardware for ML Systems (K2)

**Learning Objective AI-1.1.5:** Compare hardware choices for ML systems.

| Hardware | Cores | Parallelism | Precision | Best For |
|----------|-------|------------|-----------|----------|
| **CPU** | Few | Limited | High | Complex sequential operations |
| **GPU** | Thousands | Massive | Lower (optimized) | ML training/inference, small-scale ML |
| **ASIC/SoC** | Multiple specialized | High | Optimized | Edge computing, specific ML tasks |
| **Neuromorphic** | Brain-inspired | Novel | Variable | Emerging research, low-power AI |

**ML Hardware Requirements:** Large data structures, massively parallel processing (matrix multiplication), low-precision arithmetic (quantization — fewer bits for faster, lower-power computation).

---

### 1.1.6 Development and Hosting of AI Models (K2)

**Learning Objective AI-1.1.6:** Compare development and hosting options.

**Acquisition:** Third-party pretrained / AI-as-a-Service (AIaaS) | Private/custom development | Hybrid approaches

**Development Environments:**
- **Local:** Personal computers (small models), dedicated GPUs (mid-sized), server clusters (large-scale). Advantages: direct control, privacy. Disadvantages: hardware costs.
- **Cloud:** Public (pay-as-you-go, scalable) | Private (enhanced security) | Hybrid (prototypes locally, scale to cloud).

**Hosting:** Local (privacy, no cloud costs, limited hardware) | Cloud (scalable, powerful, maintenance-free) | Hybrid.

**Selection Factors:** Model size/complexity, performance needs, budget, security/privacy, deployment requirements, regulations.

---

### 1.1.7 ML Development Frameworks (K2)

**Learning Objective AI-1.1.7:** Summarize functionality of ML development frameworks.

A framework is a toolkit for building and training ML models, providing:

1. **Data Handling:** Loading, preprocessing, managing data
2. **Model Building:** Algorithm libraries, architecture design tools
3. **Training and Optimization:** Parameter adjustment algorithms, distributed training, fine-tuning
4. **Evaluation:** Measuring accuracy, precision, recall, error rates on unseen data
5. **Deployment:** Converting models for real-world use (web, mobile, edge)

**API Levels:** Lower-level (more control, more coding) vs Higher-level (simplified, less customization)

**Examples:** TensorFlow (Google), PyTorch (Meta), scikit-learn (general ML), Keras (high-level API)

---

### 1.1.8 Regulations and Standards for AI (K2)

**Learning Objective AI-1.1.8:** Explain how regulations and standards affect AI development/testing.

**International Soft Law:** OECD AI Principles (responsible AI stewardship), UN Governing AI for Humanity (international cooperation)

**EU AI Act** — Landmark regulation with risk-based approach:
- Risk categories from minimal to unacceptable
- High-risk systems: Rigorous testing, data governance, human oversight
- Substantial financial penalties based on global turnover

**Technical Standards:**
- **ISO/IEC TR 29119-11:** Detailed guidance on testing AI-based systems
- **ISO/IEC 42119 series:** Various aspects of AI testing (in development)
- **ISO/IEC 25059:** Quality model for AI systems
- **ISO 26262-6:** Functional safety (automotive)

---

## Chapter 2: Quality Characteristics for AI-Based Systems (45 minutes)

### Keywords
Functional adaptability, AI functional correctness, intervenability, AI robustness, safety, societal and ethical risk mitigation, transparency, user controllability

---

### 2.1.1 ISO/IEC 25059 Quality Characteristics (K2)

**Learning Objective AI-2.1.1:** Classify behaviors of AI-based systems according to ISO/IEC 25059.

ISO/IEC 25059 extends the ISO/IEC 25010 quality model with AI-specific characteristics:

**1. AI Functional Correctness** — AI cannot guarantee perfect accuracy. Defines acceptable thresholds for incorrect results reflecting inherent variability. *Example:* 95% accuracy for image recognition is acceptable; 100% is unrealistic.

**2. Functional Adaptability** — Ability to autonomously adapt to operational environment changes after deployment. *Example:* A video streaming homepage adjusts recommendations as viewing habits change.

**3. User Controllability** — Allows humans to intervene in system functioning in a timely manner, enabling immediate manual override. *Example:* An autonomous drone allows supervisor takeover on distress signal.

**4. Transparency** — Degree to which appropriate information about the AI system is communicated to stakeholders. Essential for understanding behavior and decision logic. *Example:* Dashboard showing which model version is deployed with link to documentation.

**5. AI Robustness** — Maintaining functional correctness regardless of circumstances — biased data, adversarial data, invalid inputs, external interference, adverse conditions, operator misuse. *Example:* Edge AI device transitions to lower-fidelity mode instead of crashing at extreme temperatures.

**6. Intervenability** — Degree to which an operator can intervene to prevent harm. *Example:* Power grid system provides 30-second confirmation window for engineer to veto critical AI-proposed actions.

**7. Societal and Ethical Risk Mitigation** — Mitigates risks around accountability, fairness, non-discrimination, privacy, safety, human control, environmental sustainability, and more. *Example:* Prison sentencing system doesn't discriminate between racial groups per fairness metric.

---

### 2.1.2 AI and Safety (K2)

**Learning Objective AI-2.1.2:** Explain special considerations for AI in safety-related systems.

Five challenges for AI in safety-critical systems:

1. **Specifications Challenge:** Traditional systems have complete, refined requirements. AI systems begin with vague goals, implicitly provided via training data, lacking formal detail and traceability.

2. **Non-Determinism:** Guaranteeing precise behavior is inherently difficult. Even rigorously tested models exhibit unexpected behavior due to random number generation, floating-point precision, and input variations.

3. **Self-Learning:** Rigorous pre-deployment testing is undermined as system behavior changes. Mitigation: Managing model learning and deploying content moderation guards.

4. **Explainability:** Essential to understand decision-making, but AI is often "black-box." Techniques like LIME (Local Interpretable Model-Agnostic Explanations) help but may compromise performance.

5. **Evolving Regulations:** AI not yet fully included in mature functional safety standards. Some standards explicitly prohibit AI in safety systems. EU AI Act classifies safety-critical AI as high-risk.

---

### 2.2.1 Acceptance Criteria for AI-Based Systems (K2)

**Learning Objective AI-2.2.1:** Give examples of acceptance criteria for AI-based systems.

AI acceptance criteria are often statistical, probabilistic, or threshold-based — not binary pass/fail.

**Examples by Quality Characteristic:**

| Characteristic | Example Acceptance Criterion |
|---------------|------------------------------|
| AI Functional Correctness | "Accuracy of 95% for image recognition"; "Recall of 90% for defect prediction" |
| Functional Adaptability | "Recommends 40% documentaries after 3 full-length documentaries watched" |
| User Controllability | "Notifies farmer when sensor performance degrades >30%; deactivates at >50%" |
| Transparency | "Dashboard provides unique version identifier of deployed model" |
| AI Robustness | "Response time <1 second when vulnerability DB access disrupted for 30 seconds" |
| Intervenability | "30-second confirmation window for engineer veto of critical AI actions" |
| Societal/Ethical | "Chatbot passes red teaming with 95%+ score; refuses violence/hate content" |
| Safety | "Control signals exceeding limits by >10% analyzed within 0.15 seconds" |

---

## Chapter 3: Machine Learning (375 minutes)

### Keywords
K-multisection neuron coverage, ML functional performance criteria/metric, ML model, neuron boundary coverage, neuron coverage, perceptron, association, classification, clustering, data preparation, ML algorithm, ML workflow, pretrained model, ML regression, reinforcement learning, supervised learning, unsupervised learning

---

### 3.1 Forms of Machine Learning

#### 3.1.1 ML Types (K2)

**Learning Objective AI-3.1.1:** Distinguish between different forms of ML.

**Supervised Learning** — Uses labeled data (input-output pairs):

*Classification Example — Spam Detection:*
```
Training Data:
  "Buy cheap watches now!" → Spam
  "Meeting at 3pm tomorrow" → Not Spam
  "Win a free iPhone!" → Spam
  "Project deadline extended" → Not Spam

Model learns patterns → predicts on new emails
```

*ML Regression Example — House Price:*
```
Features: Size=2000sqft, Bedrooms=3, Location=Suburban, Age=10yrs
Model predicts: $350,000 (continuous numerical value)
```

**Unsupervised Learning** — Uses unlabeled data:

*Clustering Example:* Customer purchase data grouped into segments (budget shoppers, premium buyers, occasional buyers) without predefined categories.

*Association Example:* "80% of customers who bought bread also bought butter" → product recommendation.

**Reinforcement Learning** — Agent learns through trial-and-error:
- Agent takes action in environment → receives reward/penalty → adjusts strategy
- *Example:* Robot learning to walk — rewarded for forward movement, penalized for falling.

| Aspect | Supervised | Unsupervised | Reinforcement |
|--------|-----------|-------------|--------------|
| Data | Labeled | Unlabeled | Environment + rewards |
| Goal | Predict known outputs | Discover patterns | Maximize cumulative reward |
| Types | Classification, Regression | Clustering, Association | Policy learning |
| Testing focus | Accuracy, precision, recall | Cluster quality, meaningful groupings | Reward optimization, safety |

---

#### 3.1.2 ML Workflow (K2)

**Learning Objective AI-3.1.2:** Summarize ML workflow for creating ML system.

The ML workflow consists of 11 iterative steps:

```
1. Understand Objectives → 2. Select Framework → 3. Select/Build Algorithm
       ↓                                                    ↓
4. Prepare & Test Data  ←→  EDA (Exploratory Data Analysis)
       ↓
5. Train the Model (using training dataset)
       ↓
6. Evaluate the Model (using validation dataset)
       ↓                    ↑
7. Tune the Model ──────────┘  (Steps 5-7 = "Model Generation" — iterate)
       ↓
8. Test the Model (using independent test dataset)
       ↓
9. Deploy the Model
       ↓
10. Use the Model (batch or real-time predictions)
       ↓
11. Monitor & Tune (detect drift, retrain as needed)
```

**Key Details:**

**Step 5 — Training:** Two types of hyperparameters:
- **Model hyperparameters:** Define model structure (number of layers, tree depth)
- **Algorithm hyperparameters:** Control training process (number of iterations, learning rate)

**Step 6 — Evaluation:** Uses validation dataset. Results used to improve model during tuning.

**Step 8 — Testing:** Uses independent test dataset (never seen during training). Compares with evaluation results — if significantly lower, return to model generation or data preparation.

**Step 11 — Monitoring:** Model may drift over time (see 6.1.7). Regular evaluation against acceptance criteria. A/B testing compares latest vs existing model.

---

#### 3.1.4 Pretrained Models, Fine-Tuning, RAG (K2)

**Learning Objective AI-3.1.4:** Summarize use of pretrained models, fine-tuning, RAG.

**Pretrained Models:** Training from scratch is costly and time-consuming. Use existing models as starting points.

**Fine-Tuning:** Additional training with task-specific data on a pretrained model.
- Can apply to entire network, specific layers, or additional layers
- Success depends on similarity between original and new tasks (cat breed → dog breed: highly effective; cat breed → accent recognition: less effective)
- **Caution:** Inherited biases and vulnerabilities carry over to the new model
- *Example:* Fine-tuning an LLM for ISTQB boundary value analysis is readily achievable

**RAG (Retrieval-Augmented Generation):** Provides data sources to the model at query time without modifying the model itself.
- Documents transformed into searchable format → compared with prompt → relevant documents incorporated into enhanced prompt → more precise responses
- **Advantage:** No changes to the pretrained model

**Comparison:**

| Aspect | Fine-Tuning | RAG |
|--------|------------|-----|
| Modifies model | Yes (retrains weights) | No (adds context at runtime) |
| Data needs | Labeled training data | Searchable document corpus |
| Cost | Higher (compute for training) | Lower (retrieval infrastructure) |
| Flexibility | Permanent model change | Dynamic, updateable sources |
| Best for | Domain-specific vocabulary/reasoning | Up-to-date project data, specifications |

---

### 3.2 Data Preparation

#### 3.2.1 Activities in Data Preparation (K2)

**Learning Objective AI-3.2.1:** Explain data preparation activities.

Data preparation is the most crucial and resource-intensive ML workflow activity. If operational data differs from training data, ML performance and safety assumptions may not hold.

**1. Data Acquisition:**
- Identify relevant data types: Numerical, categorical, images, text, time series, sensor, geospatial, video, audio
- Gather from diverse sources: Databases, APIs, real-time sensors
- Label data for supervised learning with verified accuracy and consistency

**2. Data Preprocessing:**
- **Cleaning:** Remove defects, duplicates, outliers; impute missing values (mean, median, mode); anonymize personal information
- **Transformation:** Format changes, scaling, normalizing for consistency
- **Augmentation:** Increase sample size, add adversarial examples for robustness, generate synthetic data
- **Sampling:** Reduce training times and computational costs

**3. Feature Engineering:**
- Select relevant features based on performance contribution
- Extract informative, non-redundant feature subset
- Reduce dimensionality for efficiency

**Parallel: Exploratory Data Analysis (EDA)** — Discover trends, patterns, anomalies using plots and charts.

#### 3.2.3 Training, Validation, and Test Datasets (K2)

**Learning Objective AI-3.2.3:** Contrast training, validation, and test datasets.

Three equivalent sets from a representative dataset:

| Dataset | Purpose | Used When |
|---------|---------|-----------|
| **Training** | Train the model | During model training (Step 5) |
| **Validation** | Evaluate and tune the model | During evaluation and tuning (Steps 6-7) |
| **Test (Holdout)** | Test the final tuned model | During testing (Step 8) — never before! |

**K-Fold Cross-Validation** (for limited data):
- Set aside small hold-out test set
- Divide remaining data into k folds (typically k=5 or 10)
- Train on k-1 folds, validate on the held-out fold
- Repeat k times (each fold serves as validation once)
- Average performance metrics across folds
- Final model trained on all data except hold-out, evaluated on hold-out

**Limitation:** Without a separate hold-out test set, cross-validation performance is optimistically biased.

---

### 3.3 ML Performance Metrics

#### 3.3.1 Confusion Matrix and Metrics (K3)

**Learning Objective AI-3.3.1:** Calculate common ML functional performance metrics from confusion matrix.

**Confusion Matrix:**

```
                    Predicted
                 Positive  Negative
Actual Positive [  TP    |   FN   ]
Actual Negative [  FP    |   TN   ]
```

- **TP (True Positive):** Correctly predicted positive
- **TN (True Negative):** Correctly predicted negative
- **FP (False Positive):** Incorrectly predicted as positive (Type I error)
- **FN (False Negative):** Incorrectly predicted as negative (Type II error)

**Four Key Metrics:**

**Accuracy = (TP + TN) / (TP + TN + FP + FN) x 100%**
Overall correctness — what proportion of ALL predictions were correct?

**Precision = TP / (TP + FP) x 100%**
Of all POSITIVE predictions, how many were actually correct? ("How reliable are positive predictions?")

**Recall (Sensitivity) = TP / (TP + FN) x 100%**
Of all ACTUAL positives, how many did we correctly identify? ("How many positives did we miss?")

**F1-Score = 2 x (Precision x Recall) / (Precision + Recall)**
Harmonic mean of precision and recall. Range 0-100. Close to 100 means both are high.

**Worked Example 1 — Cancer Screening:**

```
                 Predicted Cancer  Predicted Healthy
Actually Cancer  [    85 (TP)    |     15 (FN)     ]
Actually Healthy [    10 (FP)    |    890 (TN)     ]
Total: 1000
```

- Accuracy = (85+890)/1000 = 97.5%
- Precision = 85/(85+10) = 89.5% → "When we say cancer, we're right 89.5% of the time"
- Recall = 85/(85+15) = 85.0% → "We catch 85% of actual cancer cases"
- F1 = 2×(0.895×0.85)/(0.895+0.85) = 87.2%

*In cancer screening, Recall is critical — missing a cancer case (FN) is far worse than a false alarm (FP).*

**Worked Example 2 — Spam Detection:**

```
               Predicted Spam  Predicted Not Spam
Actually Spam  [   180 (TP)  |      20 (FN)     ]
Not Spam       [    5 (FP)   |     795 (TN)     ]
Total: 1000
```

- Accuracy = (180+795)/1000 = 97.5%
- Precision = 180/(180+5) = 97.3% → "When we say spam, we're almost always right"
- Recall = 180/(180+20) = 90.0% → "We catch 90% of spam"
- F1 = 2×(0.973×0.90)/(0.973+0.90) = 93.5%

*In spam detection, Precision matters most — marking a legitimate email as spam (FP) is highly annoying.*

---

### 3.4 Deep Neural Networks

#### 3.4.1 Structure and Working (K2)

**Learning Objective AI-3.4.1:** Explain structure and working of deep neural network.

**Perceptron:** The earliest artificial neural network — a single-layer network for binary classification of linearly separable problems. *Example:* Distinguishing spam from non-spam.

**Deep Neural Network Architecture:**
```
Input Layer      Hidden Layers (multiple)      Output Layer
  [x1] ─────→  [n1] → [n4] → [n7] ─────→  [output1]
  [x2] ─────→  [n2] → [n5] → [n8] ─────→  [output2]
  [x3] ─────→  [n3] → [n6] → [n9]
```

**Neuron Computation:**
1. Receive inputs from previous layer neurons (each connection has an independent weight)
2. Calculate weighted sum of inputs
3. Add neuron's individual bias value
4. Pass through activation function (non-linear formula)
5. Generate activation value (output to next layer)

**Training Process:**
1. Training data passes through network (forward pass)
2. Output compared with known correct result
3. Error (loss) calculated
4. Error fed back through network (backpropagation)
5. Weights and biases adjusted to minimize error
6. Each complete pass = one **epoch**
7. Training continues until output is good enough

#### 3.4.3 Neural Network Coverage Measures (K2)

**Learning Objective AI-3.4.3:** Describe coverage measures for neural networks.

Neural networks don't follow explicitly coded paths — behavior is dictated by learned weights, biases, and activations. Coverage measures assess how thoroughly test inputs exercise the model's internals.

| Measure | Definition | Granularity |
|---------|-----------|-------------|
| **Neuron Coverage** | Proportion of neurons whose output exceeds a threshold during testing | Coarse |
| **k-Multisection Neuron Coverage (kMNC)** | Each neuron's output range divided into k sections; proportion of sections activated | Fine-grained |
| **Neuron Boundary Coverage (NBC)** | Proportion of neurons whose output exceeds training maximum or falls below training minimum | Boundary exploration |

**Value:** Reveals untested areas, identifies unexercised decision boundaries, helps design additional test inputs.

**Limitations:** Structural coverage alone doesn't guarantee generalization. Networks learn spurious correlations. Must combine with adversarial testing and metamorphic testing. Limited commercial tool support currently.

---

## Practice Exercises — Chapters 1-3

**Q1 (K2).** What is the key difference between AI-based and conventional systems?
A) AI systems are faster  B) AI systems learn from data; conventional use explicit rules  C) Conventional systems use more memory  D) AI systems are more reliable

**Q2 (K2).** Which AI type does NOT currently exist?
A) Narrow AI  B) General AI  C) Frontier AI  D) Deep Learning

**Q3 (K2).** ML Regression is:
A) Same as regression testing  B) Predicting continuous numerical values  C) Testing for regressions in ML models  D) Going back to a previous model version

**Q4 (K2).** In supervised learning, classification:
A) Groups unlabeled data  B) Predicts continuous values  C) Assigns inputs to predefined categories  D) Learns through rewards

**Q5 (K2).** Which ISO standard defines AI-specific quality characteristics?
A) ISO/IEC 25010  B) ISO/IEC 25059  C) ISO/IEC 29119-11  D) ISO/IEC 42001

**Q6 (K2).** AI Robustness means:
A) The system is physically strong  B) Maintaining correctness despite adverse conditions  C) The system can handle large data  D) The system runs fast

**Q7 (K2).** The validation dataset is used to:
A) Train the model  B) Test the final model  C) Evaluate and tune the model during development  D) Deploy the model

**Q8 (K3).** Given: TP=80, TN=850, FP=20, FN=50. Calculate Accuracy:
A) 93%  B) 80%  C) 85%  D) 97%

**Q9 (K3).** Using the same data (TP=80, FP=20, FN=50), calculate Precision:
A) 62%  B) 80%  C) 94%  D) 76%

**Q10 (K3).** Using the same data, calculate Recall:
A) 62%  B) 80%  C) 94%  D) 76%

**Q11 (K2).** K-Fold Cross-Validation is used when:
A) The model is too complex  B) Data is limited  C) The model needs deployment  D) Testing in production

**Q12 (K2).** Fine-tuning a pretrained model:
A) Requires no additional data  B) Creates a completely new model  C) Adapts the model with task-specific training data  D) Only works with LLMs

**Q13 (K2).** Feature engineering involves:
A) Building hardware features  B) Selecting relevant data attributes for ML  C) Engineering new software features  D) Testing features of an application

**Q14 (K2).** Neuron Coverage measures:
A) How many neurons are in the network  B) Proportion of neurons exceeding a threshold during testing  C) Network training time  D) Number of hidden layers

**Q15 (K2).** The "technological singularity" refers to:
A) The point where AI reaches human-level intelligence  B) A specific AI algorithm  C) The transition from general AI to super AI  D) When training data becomes infinite

**Q16 (K2).** Which is a challenge of AI in safety systems?
A) AI is too expensive  B) Non-determinism makes precise behavior guarantees difficult  C) AI models are too small  D) Safety systems don't need AI

**Q17 (K2).** LIME is:
A) A machine learning algorithm  B) An explainability technique for black-box AI  C) A data augmentation method  D) A neural network architecture

**Q18 (K2).** Data augmentation in preprocessing:
A) Removes data  B) Increases sample size with synthetic/modified data  C) Labels data  D) Deploys data

**Q19 (K2).** RAG differs from fine-tuning because:
A) RAG changes the model  B) RAG provides data at query time without modifying the model  C) Fine-tuning is cheaper  D) RAG requires labeled data

**Q20 (K2).** Which hardware is generally best for small-scale ML work?
A) CPU  B) GPU  C) ASIC  D) Neuromorphic processor

**Answer Key:**
Q1:B | Q2:B | Q3:B | Q4:C | Q5:B | Q6:B | Q7:C | Q8:A (930/1000=93%) | Q9:B (80/100=80%) | Q10:A (80/130≈61.5%→62%) | Q11:B | Q12:C | Q13:B | Q14:B | Q15:C | Q16:B | Q17:B | Q18:B | Q19:B | Q20:B

---

## Additional Topics

### Transfer Learning
Using knowledge from one task to improve performance on a related task. Fine-tuning is a form of transfer learning. Critical for reducing training costs and data requirements.

### AutoML
Automated Machine Learning automates the ML workflow — algorithm selection, hyperparameter tuning, feature engineering. Implications for testing: need to validate AutoML outputs meet quality requirements.

### Federated Learning
Training across decentralized devices without sharing raw data. Preserves privacy. Testing challenges: verifying model consistency across nodes, handling non-uniform data distributions.

### Edge AI Testing
Deploying models on edge devices (phones, IoT) introduces unique testing concerns: model compression effects, hardware-specific behavior, resource constraints, offline operation.

---

## Index of Key Terms

| Term | Section |
|------|---------|
| Accuracy | 3.3.1 |
| Activation function | 3.4.1 |
| AI Functional Correctness | 2.1.1 |
| AI Robustness | 2.1.1 |
| AI-based system | 1.1.1 |
| ASIC | 1.1.5 |
| Association | 3.1.1 |
| Backpropagation | 3.4.1 |
| Bias (neural network) | 3.4.1 |
| Classification | 3.1.1 |
| Clustering | 3.1.1 |
| CNN | 1.1.3 |
| Confusion matrix | 3.3.1 |
| Cross-validation | 3.2.3 |
| Data acquisition | 3.2.1 |
| Data augmentation | 3.2.1 |
| Data preprocessing | 3.2.1 |
| Deep learning | 1.1.3 |
| Deep neural network | 3.4.1 |
| Diffusion model | 1.1.4 |
| EDA | 3.2.1 |
| Embedding | 3.1.4 |
| Epoch | 3.4.1 |
| EU AI Act | 1.1.8 |
| F1-Score | 3.3.1 |
| Feature engineering | 3.2.1 |
| Fine-tuning | 3.1.4 |
| Foundation model | 1.1.4 |
| Frontier AI | 1.1.2 |
| Functional adaptability | 2.1.1 |
| GAN | 1.1.4 |
| General AI | 1.1.2 |
| GPU | 1.1.5 |
| Hyperparameter | 3.1.2 |
| Intervenability | 2.1.1 |
| ISO/IEC 25059 | 2.1.1 |
| K-Fold Cross-Validation | 3.2.3 |
| kMNC | 3.4.3 |
| LIME | 2.1.2 |
| ML Regression | 3.1.1 |
| ML Workflow | 3.1.2 |
| Narrow AI | 1.1.2 |
| NBC | 3.4.3 |
| Neuron Coverage | 3.4.3 |
| NLP | 1.1.3 |
| Perceptron | 3.4.1 |
| Precision | 3.3.1 |
| Pretrained model | 3.1.4 |
| RAG | 3.1.4 |
| Recall | 3.3.1 |
| Reinforcement learning | 3.1.1 |
| RNN | 1.1.3 |
| Super AI | 1.1.2 |
| Supervised learning | 3.1.1 |
| Technological singularity | 1.1.2 |
| Training dataset | 3.2.3 |
| Transformer | 1.1.3 |
| Transparency | 2.1.1 |
| Unsupervised learning | 3.1.1 |
| User controllability | 2.1.1 |
| Validation dataset | 3.2.3 |

## References

1. ISTQB CT-AI Syllabus v2.0 (2026)
2. ISO/IEC 25059:2023 — Quality model for AI systems
3. ISO/IEC 25010:2023 — Product quality model
4. ISO/IEC TR 29119-11:2020 — Testing of AI-based systems
5. ISO 26262-6:2018 — Functional safety (automotive)
6. EU AI Act — Regulation (EU) 2024/1689
7. OECD AI Principles (2024)
8. ISTQB Foundation Level Syllabus v4.0

---

*Continue to Part 2 for Chapters 4-7: Testing AI Systems, Input Data Testing, Model Testing, Development Testing*
