# ISTQB CT-AI Study Guide — Part 2: Testing AI-Based Systems

> **Based on:** ISTQB Certified Tester AI Testing (CT-AI) Syllabus v2.0
> **Chapters Covered:** 4, 5, 6, and 7
> **Total Study Time:** 195 + 180 + 225 + 30 = 630 minutes (~10.5 hours)

---

## Chapter 4: Testing AI-Based Systems (195 minutes)

### Keywords
Attack, exploratory testing, risk-based testing, test oracle, adaptive AI-based system, locked AI-based system, generative AI, large language model

---

### 4.1.1 Locked vs Adaptive AI-Based Systems (K2)

**Learning Objective AI-4.1.1:** Compare testability of locked and adaptive AI-based systems.

**Locked AI-Based Systems** do not change behavior once deployed. The model's weights and biases are fixed — it can only be changed if retrained and redeployed.

*Example:* A deployed DNN for lane detection in self-driving cars. The model remains constant between updates.

**Testing Advantage:** Largely deterministic post-deployment; expected results don't change; a new version is treated as a new system requiring fresh testing.

**Note:** Even locked systems show minor non-determinism due to floating-point precision limits, hardware execution variations, and parallel computation on GPUs.

**Adaptive AI-Based Systems** adapt behavior once deployed based on new data, user interactions, or reward functions.

*Example:* An e-commerce recommendation engine that learns from customer purchases. A reinforcement learning system adjusting its strategy based on operational rewards.

**Testing Challenges:**
- System may change during or as a result of testing
- New behaviors cannot be predicted or tested in advance
- Cannot prepare test cases for unpredictable behaviors

**Testing Approaches:**
- Pre-deployment: Simulate environment changes, test learning mechanism and adaptation in controlled scenarios
- Post-deployment: Automated test suite for core functionality executed when system undergoes significant changes; monitor performance against degradation thresholds

**GenAI Systems (LLM-based):** Deployed as locked models at runtime, updated periodically. They sit between fully locked and fully adaptive.

**Spectrum:**
```
Fully Deterministic ←─────────────────────→ Fully Self-Learning
    Locked DNN        GenAI/LLM        RL Agent      Adaptive System
    (fixed weights)   (periodic updates) (learns from   (continuous
                                         rewards)       adaptation)
```

---

### 4.1.2 Statistical Approach to Testing (K2)

**Learning Objective AI-4.1.2:** Explain why a statistical approach is needed for AI-based system testing.

**Why a Single Test Is Insufficient:**

1. **Non-Determinism:** AI systems are fundamentally probabilistic. The same input may produce different outputs. A single misclassification (cat identified as dog) doesn't reflect overall model quality. Solution: Large, statistically significant test suites.

2. **Distributional Performance:** Models are trained on specific data distributions, but real-world data doesn't exactly match. Must test with a statistically significant sample from operational data distributions.

3. **Handling Uncertainty and Bias:** AI systems produce confident but potentially incorrect predictions. Statistical testing quantifies accuracy, fairness, and robustness using confidence intervals, hypothesis testing, and error analysis.

4. **Regulatory and Safety Requirements:** Regulated industries must demonstrate safety and fairness thresholds with high confidence using statistical methods, not just individual examples.

*Example:* An image classifier achieves 95% accuracy on 10,000 test images. This is a statistically meaningful result. Testing with 5 images and getting 4/5 correct (80%) is meaningless due to insufficient sample size.

---

### 4.1.3 Test Oracles for AI-Based Systems (K2)

**Learning Objective AI-4.1.3:** Explain challenges and solutions for AI-based system test oracles.

**Challenges:**
1. **Probabilistic Nature:** Outputs vary; need tolerance ranges instead of exact matches
2. **Incomplete Specifications:** AI development is exploratory; requirements evolve
3. **Task Complexity:** Tasks too complex for straightforward human verification
4. **Subjectivity:** "Correct" behavior may be subjective (virtual assistant quality)
5. **Self-Learning:** Correct behavior changes over time as the system adapts

**Solutions:**
1. **Output Boundaries:** Acceptable ranges, distributions, tolerances. *Example:* Autonomous car must stop within maximum distance ± tolerance.
2. **Environmental Boundaries:** Specify test conditions (lighting, temperature, latency).
3. **Expert Consultation:** Domain experts define expected results (may disagree).
4. **Specialized Approaches:** A/B testing, back-to-back testing, metamorphic testing — assess without explicit expected outputs.
5. **Proxy Oracles:** Secondary systems/models validate outputs. *Example:* Train a proxy model on labeled data to predict for unlabeled tests.

---

### 4.2.1 Testing Generative AI (K2)

**Learning Objective AI-4.2.1:** Explain how generative AI can be tested.

Testing GenAI evaluates correctness, coherence, creativity, originality, and requirement conformance.

**Black-Box Approach:** Feed various inputs (prompts, images, partial data), assess results manually or automatically considering clarity, originality, and domain-specific rules.

**Input Explosion Problem:** Inputs are highly diverse — system prompts, user prompts, additional data, parameters (temperature, max tokens), context window history — making exhaustive testing impossible.

**Output Assessment:** Manual review is common. Pass/fail depends on qualitative evaluation criteria. Alternative: second GenAI system automatically evaluates (caution: may reproduce similar biases).

**Non-Functional Testing:** Evaluate CPU/GPU usage, memory, network bandwidth, response times during training and inference. Verify cost-effectiveness.

**Benchmarking:** Standardized evaluation frameworks with curated datasets for consistent cross-model comparison of language understanding, reasoning, and coding abilities.

---

### 4.2.2 Red Teaming (K3)

**Learning Objective AI-4.2.2:** Implement red teaming for GenAI systems.

Red teaming is systematic, often black-box fault attack probing to identify harmful capabilities in AI outputs — privacy violations, racist views, dangerous guidance.

**Security Red Teaming:** Identifies vulnerabilities to external attacks (prompt injection, malicious content in RAG documents). Focuses on malicious inputs.

**Safety Red Teaming:** Identifies harmful outputs under ordinary (non-malicious) use — unsafe medical advice, misinformation. Broader scope than security.

**Five-Step Process:**
1. Assemble diverse tester teams (wide perspective coverage)
2. Provide safe test environment access
3. Probe system to identify vulnerabilities (open-ended or checklist-based)
4. Analyze identified failures to understand threats
5. Create datasets from threats for mitigations/improvements

**Core Activity:** Multi-turn dialogues (15-20 turns) eliciting defective or policy-violating behaviors.

**Coverage Strategies:**
- **Manual:** Crowd-sourced prompt generation from diverse testers
- **Automated:** LLM generates attack prompts; another LLM checks outputs
- **Hybrid:** Human creativity + automation scalability

**Blue Teaming:** Complementary ongoing defensive monitoring/filtering protecting operational AI against attacks. Red Teaming insights enhance blue team filters.

---

### 4.3.1 Test Levels for ML Systems (K2)

**Learning Objective AI-4.3.1:** Summarize test levels for ML systems.

**Conventional Test Levels** (for non-AI components):
- Component Testing: UI, data pipeline, communication components
- Component Integration Testing: Data pipeline ↔ model interactions; API testing for AI-as-a-service
- System Testing: Verify ML performance isn't degraded when embedded in complete system
- System Integration Testing: External system interfaces
- Acceptance Testing: Verify ML functional performance criteria

**ML-Specific Test Levels:**
- **Input Data Testing** (Chapter 5): Training data and production data quality
- **ML Model Testing** (Chapter 6): ML model quality and performance

---

### 4.3.2 Risk-Based Testing of ML Systems (K2)

**Learning Objective AI-4.3.2:** Explain how risk-based testing applies to ML systems.

Three risk categories by ML workflow area:

| Category | Focus | Example Risks |
|----------|-------|---------------|
| **Development** | Algorithm, model, framework | Sub-optimal algorithm, framework vulnerabilities, poor evaluation |
| **Input Data** | Training and production data | Biased data, pipeline defects, unrepresentative data |
| **Model** | Generated ML model | Poor performance, overfitting, adversarial susceptibility |

Alternative categorization: By ISO/IEC 25059 quality characteristics.

---

## Chapter 5: Input Data Testing (180 minutes)

### Keywords
Data pipeline testing, data representativeness testing, dataset constraint testing, input data testing, label correctness testing, review, testing for bias, disparate impact analysis, multiple annotation

---

### 5.1.1 Input Data Risks and Mitigations (K2)

**Learning Objective AI-5.1.1:** Give examples of test approaches for input data risk mitigation.

| Risk | Mitigation |
|------|-----------|
| Biased training data | Testing for bias (5.1.2) |
| Untrustworthy data sources | Data provenance testing |
| Poorly managed data | EDA (3.2.1) |
| Poisoned training data | A/B testing (6.1.9); Red teaming (4.2.2) |
| Inconsistent/out-of-range data | Dataset constraint testing (5.1.5) |
| Sub-optimal features | Feature testing |
| Imbalanced/skewed dataset | Data representativeness testing (5.1.4) |
| Poor labeling | Label correctness testing (5.1.6) |
| Pipeline design/integration issues | Data pipeline testing (5.1.3) |

---

### 5.1.2 Testing for Bias (K2)

**Learning Objective AI-5.1.2:** Explain how to test for bias.

**Bias** = Non-random, unfair differences based on sensitive attributes (gender, age, race); often unlawful and discriminatory.

**Sources:** Data bias (defects in training data) and algorithmic bias (defects in algorithm/model/framework).

**Seven Test Approaches:**

1. **Reviews of ML Workflow** — especially data preparation, for bias potential
2. **Dataset Documentation Reviews** — examine collection, annotation, represented populations
3. **Static Analysis** — code analysis for anti-patterns and sensitive attribute mishandling
4. **EDA of Training Data** — visualization revealing imbalanced/skewed distributions
5. **Dynamic Testing** — feed known unbiased dataset, analyze for statistically significant differences across sensitive groups
6. **Label Correctness Testing** — identify mislabeling causing incorrect associations
7. **Disparate Impact Analysis** — systematic four-step process:

**Disparate Impact Analysis (Detailed):**

*Step 1:* Identify sensitive attributes for the model (gender, race, age, etc.)

*Step 2:* Generate counterfactuals — change only the sensitive attribute.
*Example:* Loan application with `Gender: Male` → create identical application with `Gender: Female`

*Step 3:* Generate model output with counterfactuals

*Step 4:* Analyze multiple test results for statistical significance. If changing the sensitive attribute significantly changes the model output → bias detected.

*Extended:* Apply to attribute combinations (gender + race) for hidden bias.

*Caution:* Ensure counterfactuals are realistic — the model should respond to bias, not to an unrealistic data combination.

---

### 5.1.3 Data Pipeline Testing (K2)

**Learning Objective AI-5.1.3:** Summarize forms of data pipeline testing.

Layered approach through all test levels:

| Level | Focus |
|-------|-------|
| **Design Reviews** | Pipeline architecture quality |
| **Component Testing** | Ingestion, transformation scripts, sensors. Code reviews, static analysis |
| **Component Integration** | Seamless data flow across internal interfaces |
| **System Testing** | Smoke testing, functional testing, non-functional (performance/security), fault injection, back-to-back |
| **System Integration** | Interaction with external systems/data sources |
| **Production** | Back-to-back (vs. previous versions), A/B testing, continuous monitoring |

**Training vs Operational Pipelines:** Training pipelines are often exploratory prototypes focused on data integrity. Operational pipelines prioritize reliability, performance, and maintainability.

---

### 5.1.4 Data Representativeness Testing (K2)

**Learning Objective AI-5.1.4:** Explain how to test for data representativeness.

**Three-Step Process:**

**Step 1: Define Target Population**
- Understand use cases and operational context
- Identify expected data distributions and critical edge cases
- Consult domain experts and benchmark datasets
- Apply stratified sampling to create a representative baseline

**Step 2: Analyze Data Characteristics**
- Apply EDA to training/test datasets AND reference dataset
- Visualize distributions (histograms, scatter plots)
- Examine feature relationships and correlations
- Identify gaps, anomalies, unusual concentrations

**Step 3: Statistical Assessment**
- Formal tests: Chi-squared, Kolmogorov-Smirnov
- Check for class imbalances
- Verify edge/boundary case coverage

**Timing:** Before model training. After deployment, continuously monitor for data drift.

---

### 5.1.5 Dataset Constraint Testing (K3)

**Learning Objective AI-5.1.5:** Apply dataset constraint testing.

Constraints define the "logical model" for ML datasets. Verify data adheres to predefined rules:

**Single-Value Constraints (one attribute, one instance):**

| Type | Test | Example |
|------|------|---------|
| **Missing** | Value/attribute present? | Name field must not be null |
| **Range** | Value within bounds? | Age must be 0-150 |
| **Type** | Correct data type? | Age must be integer, not string |

**Multi-Value Constraints (one attribute, multiple instances):**

| Type | Test | Example |
|------|------|---------|
| **Sum** | Total within bounds? | F1 race points cannot exceed 102 |
| **Count** | Number of non-null values within bounds? | At least 500 non-null records required |
| **Duplicate** | Allowable duplicates? | No duplicate customer IDs |
| **Useful** | Sufficient variety? | If all values are unique (like timestamps), no useful ML patterns |
| **Outlier** | Statistical outliers present? | Values >3 standard deviations from mean |

**Comparison Constraints (comparing different values):**

| Type | Test | Example |
|------|------|---------|
| **Greater Than** | One value > another? | Lines of code > lines with defects |
| **Correlate** | Values correlate? | Students with marks 1.33σ above mean should have 'A' grade |

**Implementation:** Typically automated as part of data pipeline. Reports sent to data scientists (training anomalies) or operations (production problems).

---

### 5.1.6 Label Correctness Testing (K2)

**Learning Objective AI-5.1.6:** Explain label correctness testing.

Essential for supervised learning — inaccurate/inconsistent labels undermine performance.

**Seven Approaches:**

1. **Expert Review:** Domain experts manually review labeled data samples
2. **Multiple Annotation:** Multiple annotators independently label same data points; compare using **Inter-Annotator Agreement (IAA)** measured with Cohen's Kappa. Low IAA → guideline defects or ambiguous data
3. **Risk-Based Prioritization:** Focus reviews on samples most likely mislabeled or most impactful
4. **Data Distribution Analysis:** Compare label distributions to similar datasets; reveal anomalies
5. **Automated Rule-Based Tests:** Predefined constraints. *Example:* Verify bounding boxes don't overlap or extend beyond image boundaries
6. **Model Loss Analysis:** High loss during training indicates potential mislabeling
7. **Model Confidence Score Analysis:** Low prediction confidence suggests mislabeling, ambiguity, or out-of-distribution data

**Best Practice:** Combine expert reviews (clear guidelines early), IAA scores (refine labeling process), and model-based approaches (identify label defects as model develops).

---

## Chapter 6: Model Testing (225 minutes)

### Keywords
A/B testing, adversarial testing, back-to-back testing, concept drift, data drift, drift testing, metamorphic testing, ML functional performance, ML model testing, review, overfitting, underfitting

---

### 6.1.1 ML Model Risks and Mitigations (K2)

**Learning Objective AI-6.1.1:** Give examples of test approaches for ML model risk mitigation.

| Risk | Mitigation |
|------|-----------|
| Biased/unfair model | Testing for bias (5.1.2) |
| Adversarial vulnerability | Adversarial testing (6.1.4) |
| Overfitted model | Testing for overfitting (6.1.8) |
| Underfitted model | Testing for underfitting (6.1.8) |
| Data/concept drift | Drift testing (6.1.7) |
| ML performance failure | ML functional performance testing (6.1.3) |
| Functional incorrectness | Metamorphic testing (6.1.5); Back-to-back (6.1.10) |
| Model updates introduce defects | Back-to-back testing (6.1.10) |
| Updates decrease performance | A/B testing (6.1.9) |
| Security/safety vulnerabilities | Red teaming (4.2.2); Exploratory testing |

---

### 6.1.2 ML Model Documentation and Review (K2)

**Learning Objective AI-6.1.2:** Explain purpose and focus of reviewing ML model documentation.

Documentation is the main tool for developers, testers, and regulators to understand, evaluate, and trust AI systems. Critical for regulatory compliance (EU AI Act emphasizes transparency).

**Documentation Frameworks:**
- **Model Cards:** Concise overviews, intended uses, evaluation results, ethical considerations
- **Datasheets for Datasets:** Standardized dataset descriptions (motivation, composition, collection, uses)

**Typical Documentation Checklist:**
- General: Identifiers, description, developer, version, license, hardware needs
- Design: Assumptions, technical decisions, ML algorithm
- Usage: Intended purpose, users, bias, ethics, safety, transparency
- Datasets: Features, source, preprocessing, labels, size, privacy, bias
- Testing: Test dataset, independence, results, activities (adversarial, etc.)
- Functional: Performance measures, thresholds, actual results
- Non-Functional: Scalability, reliability, performance efficiency, robustness
- Operational: Deployment plan, monitoring, retraining strategy, rollback plan

---

### 6.1.3 ML Functional Performance Testing (K2)

**Learning Objective AI-6.1.3:** Explain ML functional performance testing for probabilistic systems.

Acceptance criteria must be statistically defined with three components: **Performance metric + Margin of Error (MoE) + Confidence Level (CL)**.

*Example:* "98% accuracy with MoE of ±4% at 95% CL"

**Sample Size Calculation (Illustrative):**
- For the above criteria: approximately 601 test cases required
- To meet target: at least 589 of 601 must pass
- Safety-critical: "99% reliability at 95% CL" → 299 test cases, ALL must pass

**Execution Approaches:**
- **Fixed Sample Size:** Run all test cases, calculate final accuracy/MoE
- **Sequential Testing (Preferred):** Analyze as results accumulate, stop early when sufficient evidence exists (e.g., ~170 test cases if consistently ≥98%)

**Test Dataset Requirements:** Completely independent from training/validation; representative of operational input domain.

**Reporting:** "Model achieved 94% accuracy ±4% at 95% CL" — stakeholders understand expected operational performance range.

---

### 6.1.4 Adversarial Testing (K2)

**Learning Objective AI-6.1.4:** Summarize adversarial testing of ML systems.

Deliberately provide models with input perturbations (often imperceptible to humans) to cause incorrect predictions. Successful perturbations = **adversarial examples**.

**Black-Box Approach:** Create an equivalent model with known internals, generate adversarial inputs from that knowledge (transferability assumption), apply to original model.

**White-Box Approach:** Use knowledge of model internals (architecture, parameters) to craft targeted adversarial examples. Typically easier.

**Implementation:** Manual crafting of specific examples OR automated algorithms generating large variations.

*Example:* Adding imperceptible noise to a stop-sign image causes a self-driving car's classifier to read it as "Speed Limit 45."

---

### 6.1.5 Metamorphic Testing (K3)

**Learning Objective AI-6.1.5:** Use metamorphic testing to derive test cases.

Metamorphic testing derives **follow-up test cases** from previously passed **source test cases** using **metamorphic relations (MRs)** — rules describing how input changes should reflect in output changes.

**Verification Objectives:**
- **Consistency:** Outputs align across related inputs
- **Monotonicity:** Outputs change directionally with input
- **Invariance:** Outputs remain stable under perturbations

**When to Use:** No reliable expected outputs (model opacity, data scale), black-box system, relational properties sufficient for confidence.

**Worked Example — Life Expectancy Model:**

Source test case: Person who smokes 10 cigarettes/day → predicted life expectancy = 72 years

Metamorphic Relation (Monotonicity): Increasing cigarette count should decrease predicted life expectancy.

Follow-up test case: Same person, 20 cigarettes/day → predicted life expectancy should be < 72 years

If the follow-up prediction is 75 years → MR violated → defect detected.

**Worked Example — Image Classifier:**

Source test case: Image of cat → classified as "cat" (passed)

MR (Invariance): Rotating the image slightly should not change the classification.

Follow-up: Rotate cat image 5 degrees → should still classify as "cat"

If classified as "dog" → MR violated → defect detected.

**Deriving MRs:** From domain knowledge, requirements, physical laws. Validate via expert review.

**Limitations:** Incorrect MRs lead to false confidence. Detects relational flaws, not absolute errors. Must combine with other techniques.

---

### 6.1.7 Drift Testing (K2)

**Learning Objective AI-6.1.7:** Explain drift testing for operational ML systems.

**Data Drift:** Statistical properties of input data change over time — data now differs significantly from training data. *Examples:* User behavior shifts, seasonality, new phishing attack types not in training.

**Concept Drift:** The relationship between input and correct output changes — learned patterns no longer reflect reality. *Example:* New financial regulations make previously "low-risk" transactions now "high-risk."

**Testing Approaches:**

| Approach | Requires Ground Truth? | Method |
|----------|----------------------|--------|
| **Dynamic** | Yes (direct or indirect user feedback) | Compare current ground truth with model output against threshold |
| **Static** | No | Compare input/output data distributions over time (Kolmogorov-Smirnov test) |

---

### 6.1.8 Overfitting and Underfitting (K2)

**Learning Objective AI-6.1.8:** Explain how overfitting/underfitting are detected by testing.

**Overfitting:** Model learns training data too well (including noise), performing poorly on new data.

```
Detection: Test performance << Validation performance
Learning Curve:
  Validation error: ───────── (low)
  Test error:       ─────────────── (much higher)
                    → Big gap = overfitting
```

**Underfitting:** Model too simple to capture underlying patterns, performing poorly on everything.

```
Detection: Low metrics on BOTH training AND validation
Learning Curve:
  Training error:    ─────────── (high)
  Validation error:  ─────────── (high, close to training)
                    → Both high, close together = underfitting
```

**Right-Fitting:** The goal — good performance on both training/validation and test data.

---

### 6.1.9 A/B Testing (K2)

**Learning Objective AI-6.1.9:** Explain A/B testing in ML system context.

A/B testing compares two program variants (A and B) using the same inputs and statistical techniques to determine which is better.

**ML Application:** Whenever a system is updated, A/B testing determines whether the updated variant performs as well or better than the previous version.

*Example:* Smart city transport routing update — compare average commute times for variant A (old) vs variant B (new) on consecutive weeks.

**For Self-Learning Systems:** Automated tests run before/after system changes. If improved → accept; otherwise → revert.

**Statistical Techniques:** t-test, z-test, chi-squared, Mann-Whitney U test.

**Distinction from Back-to-Back Testing:** A/B compares using ML performance metrics statistically. Back-to-back testing detects specific defects.

---

### 6.1.10 Back-to-Back Testing (K2)

**Learning Objective AI-6.1.10:** Explain back-to-back testing in ML system context.

Uses an alternative system version as a **pseudo-oracle** — compare its outputs with the system under test for the same inputs.

**Key Principle:** Pseudo-oracle and system under test should NOT share common software components (both might have the same defect). Use different frameworks, algorithms, model settings, or even conventional software solving the same problem.

**Advantages:** Only needs test inputs (not expected results). Large test numbers possible with automation. Particularly valuable for migration, deployment comparison, and edge case detection.

---

## Chapter 7: ML Development Testing (30 minutes)

### Keywords: ML development testing, ML functional performance, shadow testing

---

### 7.1.1 Development Risks and Mitigations (K2)

**Learning Objective AI-7.1.1:** Give examples of test approaches for ML development risk mitigation.

| Risk | Mitigation |
|------|-----------|
| Incorrect API use (TensorFlow, PyTorch) | API testing |
| Sub-optimal framework | Framework suitability review |
| Framework security vulnerabilities | Security testing |
| Defective algorithm implementation | ML performance testing; Back-to-back testing |
| Sub-optimal algorithm/hyperparameters | A/B testing; ML performance testing |
| Deployment defect | Smoke testing; ML performance testing |
| Incompatible with operational environment | Deployment testing (7.1.2) |
| No improvement over current model | Shadow testing (7.1.2) |

---

### 7.1.2 ML System Deployment Testing (K2)

**Learning Objective AI-7.1.2:** Explain various forms of ML system deployment testing.

| Test Type | Purpose | Key Detail |
|-----------|---------|------------|
| **Installability** | Verify MLS installs/configures/uninstalls correctly | All environments, GPU drivers, framework compatibility |
| **Rollback** | Verify reversion to previous stable state | Must test BEFORE deployment |
| **Canary** | Release to small traffic subset (e.g., 5%) | Monitor latency, accuracy, errors in real-time |
| **Shadow** | Run new model parallel with production (same inputs, no live impact) | Compare outputs for regressions, drift |
| **Model Conversion** | Verify model retains accuracy after format conversion | Check inference speed, memory usage |
| **Cross-Device** | Verify across intended targets (mobile, edge, cloud) | Hardware-specific behavior |
| **API** | Verify well-defined, standards-compliant interfaces | Input/output handling, error messages, integration |

---

## Practice Exercises — Chapters 4-7

**Q1 (K2).** A locked AI system:
A) Adapts after deployment  B) Does not change behavior once deployed  C) Only works offline  D) Cannot be retested

**Q2 (K2).** Why is statistical testing necessary for AI systems?
A) AI systems are expensive  B) Single test instances don't reflect overall correctness  C) AI always gives wrong answers  D) Regulations require exactly 1000 tests

**Q3 (K2).** A proxy oracle is:
A) A human expert  B) A secondary system used to validate AI outputs  C) The original specification  D) A debugging tool

**Q4 (K3).** In red teaming, the core interactive activity involves:
A) Single-prompt tests  B) Multi-turn dialogues (15-20 turns)  C) Automated scans only  D) Performance benchmarks

**Q5 (K2).** Disparate impact analysis involves:
A) Measuring response time  B) Changing sensitive attributes and comparing outputs  C) Testing with random data  D) Measuring energy consumption

**Q6 (K3).** A dataset constraint "Type" check verifies:
A) Data is within a range  B) No missing values  C) Attribute values match specified data type  D) No duplicates

**Q7 (K2).** Low Inter-Annotator Agreement indicates:
A) Labels are perfect  B) Labeling guidelines are defective or data is ambiguous  C) The model is accurate  D) More data is needed

**Q8 (K2).** "98% accuracy ±4% at 95% CL" means:
A) The model is 98% accurate  B) We're 95% confident the true accuracy is between 94-100%  C) 98% of tests pass  D) 4% of data is missing

**Q9 (K3).** In metamorphic testing, a metamorphic relation is:
A) The expected output  B) A rule describing how input changes should affect output changes  C) A test oracle  D) A statistical formula

**Q10 (K2).** Data drift occurs when:
A) The model overfits  B) Input data statistics change over time  C) The model is retrained  D) Labels are incorrect

**Q11 (K2).** Overfitting is detected when:
A) Training error is high  B) Test performance is much worse than validation performance  C) Both training and test error are high  D) The model is too complex

**Q12 (K2).** Shadow testing:
A) Tests in darkness  B) Runs new model parallel with production without affecting live responses  C) Tests backup systems  D) Tests after deployment failure

**Q13 (K2).** Back-to-back testing detects defects by:
A) Running the same test twice  B) Comparing outputs of two different systems given the same inputs  C) Testing front and back end separately  D) Using A/B statistical analysis

**Q14 (K2).** Concept drift means:
A) Data formatting changes  B) The relationship between input and correct output changes over time  C) The model moves to a new server  D) Features are removed

**Q15 (K2).** Canary testing releases to:
A) All users  B) A small subset of production traffic  C) Only test environments  D) Only internal users

**Q16 (K3).** Which metamorphic relation tests invariance?
A) Increasing input should increase output  B) Slightly rotating an image should not change classification  C) Doubling input should double output  D) Removing input should remove output

**Q17 (K2).** Blue teaming complements red teaming by:
A) Attacking the system  B) Providing ongoing defensive monitoring  C) Training the AI  D) Generating test data

**Q18 (K2).** Model Cards are:
A) Physical cards for model identification  B) Concise model documentation including uses, evaluation, ethics  C) Credit cards for API billing  D) Test case templates

**Q19 (K2).** Sequential testing is preferred over fixed sample size because:
A) It uses fewer resources  B) It can stop early when sufficient evidence exists  C) It's more accurate  D) It doesn't need a test dataset

**Q20 (K2).** In adversarial testing, transferability means:
A) Moving models between servers  B) Adversarial examples effective on one model often work on similar models  C) Test cases can be reused  D) Data can be transferred securely

**Answer Key:**
Q1:B | Q2:B | Q3:B | Q4:B | Q5:B | Q6:C | Q7:B | Q8:B | Q9:B | Q10:B | Q11:B | Q12:B | Q13:B | Q14:B | Q15:B | Q16:B | Q17:B | Q18:B | Q19:B | Q20:B

---

## Additional Topics

### MLOps Testing Pipeline
Continuous testing integrated into the ML lifecycle: automated data validation, model evaluation on each training run, performance monitoring, drift detection, and automated rollback triggers.

### Chaos Engineering for AI
Deliberately introducing failures (corrupted inputs, infrastructure outages, resource constraints) to test AI system resilience. Borrows from software chaos engineering but applies to ML-specific failure modes.

### Explainable AI (XAI) Testing
Testing the explanations AI systems provide for their decisions. Techniques include SHAP (SHapley Additive exPlanations) and LIME values, verifying they are consistent, meaningful, and not misleading.

### Model Compression Testing
Testing models after pruning (removing unimportant weights), quantization (reducing numerical precision), or knowledge distillation (training smaller model from larger). Verify acceptable performance retention.

### Synthetic Data Generation
Using generative models to create training/test data that preserves statistical properties without exposing real data. Testing involves verifying representativeness and utility.

---

## Index of Key Terms

| Term | Section |
|------|---------|
| A/B testing | 6.1.9 |
| Adaptive AI system | 4.1.1 |
| Adversarial testing | 6.1.4 |
| Back-to-back testing | 6.1.10 |
| Blue teaming | 4.2.2 |
| Canary testing | 7.1.2 |
| Concept drift | 6.1.7 |
| Cross-device testing | 7.1.2 |
| Data drift | 6.1.7 |
| Data pipeline testing | 5.1.3 |
| Data representativeness | 5.1.4 |
| Dataset constraint testing | 5.1.5 |
| Disparate impact analysis | 5.1.2 |
| Drift testing | 6.1.7 |
| Exploratory testing | 4.2.3 |
| Input data testing | 5.1.1 |
| Label correctness | 5.1.6 |
| Locked AI system | 4.1.1 |
| Metamorphic relation | 6.1.5 |
| Metamorphic testing | 6.1.5 |
| ML functional performance | 6.1.3 |
| ML model testing | 6.1.1 |
| Model Cards | 6.1.2 |
| Model conversion testing | 7.1.2 |
| Multiple annotation | 5.1.6 |
| Overfitting | 6.1.8 |
| Proxy oracle | 4.1.3 |
| Red teaming | 4.2.2 |
| Risk-based testing | 4.3.2 |
| Rollback testing | 7.1.2 |
| Sequential testing | 6.1.3 |
| Shadow testing | 7.1.2 |
| Statistical testing | 4.1.2 |
| Test oracle | 4.1.3 |
| Underfitting | 6.1.8 |

## References

1. ISTQB CT-AI Syllabus v2.0 (2026)
2. ISO/IEC 25059:2023 — Quality model for AI systems
3. ISO/IEC TR 29119-11:2020 — Testing of AI-based systems
4. ISO/IEC TS 42119-2:2025 — Testing of AI overview
5. EU AI Act — Regulation (EU) 2024/1689
6. ISTQB Foundation Level Syllabus v4.0

---

*End of CT-AI Study Guide. Combined with Part 1, this covers the complete CTAI v2.0 syllabus.*
