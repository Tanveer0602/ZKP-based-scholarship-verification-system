# 🎓 Scholarship Decision Model with Zero-Knowledge Proofs (EZKL)

This project demonstrates how to build a **privacy-preserving scholarship eligibility model** using [EZKL](https://github.com/zkonduit/ezkl), a framework for generating zero-knowledge proofs of AI model inference. We implement a simple rule-based model and create verifiable proofs that a candidate meets the scholarship criteria—**without revealing sensitive information**.

---

##  Project Overview

* **Objective**: Prove a decision (approve/reject scholarship) based on GPA, income, and test score, **without revealing the actual inputs**.
* **ZK Use-case**: Use zero-knowledge proofs to verify AI model inference.
* **Framework**: [EZKL](https://github.com/zkonduit/ezkl) (ZKP over ONNX models)

---

## 📁 Folder Structure

```bash
.
├── model.onnx          # ONNX-exported model
├── settings.json       # Circuit compilation config
├── input.json          # Input values to test
├── model.compiled      # Compiled ZK circuit
├── witness.json        # Intermediate execution trace
├── pk.key              # Proving key
├── vk.key              # Verification key
├── kzg.srs             # Structured Reference String (SRS)
├── proof.json          # Final ZK proof
└── README.md           # This file
```

---

##  Step-by-Step Workflow

### 1. Define & Export the Model

We use a simple PyTorch model implementing business rules:

```python
approved = (gpa >= 8.5) & (income <= 100000) & (score >= 85)
```

**Export to ONNX**:

```python
torch.onnx.export(model, example_input, "model.onnx", ...)
```

---

### 2. Create `settings.json`

```json
{
  "run_args": {
    "input_visibility": "public",
    "output_visibility": "public"
  },
  "tolerance": {
    "val": 0,
    "type": "abs"
  }
}
```

* `input_visibility`: whether inputs are public or private.
* `tolerance`: acceptable precision (used for floating-point ops).

---

### 3. Compile the Model

Compile the ONNX model into a ZK circuit:

```bash
ezkl compile-circuit -M model.onnx --settings-path settings.json
```

🔹 Output: `model.compiled`

---

### 4. Create the Input File

Example input:

```json
{
  "input_data": [[9.2, 95000, 92]]
}
```

Save it as `input.json`.

---

### 5. Generate Witness

Compute intermediate values of model inference:

```bash
ezkl gen-witness --data input.json --compiled-circuit model.compiled --output witness.json
```

🔹 Output: `witness.json`

---

### 6. Generate SRS (Trusted Setup)

```bash
ezkl gen-srs --srs-path kzg.srs --logrows 17
```

* `kzg.srs`: Needed for cryptographic operations like commitments.

---

### 7. Setup Keys

```bash
ezkl setup --compiled-circuit model.compiled --vk-path vk.key --pk-path pk.key --srs-path kzg.srs
```

* `vk.key`: For verifying proofs
* `pk.key`: For generating proofs

---

### 8. Generate the ZK Proof

```bash
ezkl prove --witness witness.json --compiled-circuit model.compiled \
--pk-path pk.key --proof-path proof.json --srs-path kzg.srs
```

🔹 Output: `proof.json`
This file **proves the model ran correctly on your input**—without revealing that input.

---

### 9. Verify the Proof (Optional)

```bash
ezkl verify --proof-path proof.json --compiled-circuit model.compiled \
--vk-path vk.key --srs-path kzg.srs
```

If successful, this proves the decision (e.g. scholarship approved) is valid and **trusted**.

---

## 🔐 Use-Case Significance

* **Privacy**: No need to reveal GPA, income, or score.
* **Verifiability**: Third parties can verify eligibility.
* **Modularity**: You can update rules without touching the ZK system.

---

## 📚 Dependencies

* [Rust + Cargo](https://www.rust-lang.org/tools/install)
* [EZKL v7.2.0](https://github.com/zkonduit/ezkl)
* Python (for model creation)
* PyTorch (for model export)

---

## 🤝 Contributing

Feel free to fork this repo and extend it with:

* Real ML models
* Private inputs
* EVM integration

---

##  Future Work

* EVM verifier generation
* Integration with web frontend
* Multi-party setup (MPC ceremony)

---

Let me know if you'd like to include a [badge-based summary](f), a [GitHub Actions pipeline](f), or steps for [deploying to EVM](f)!
