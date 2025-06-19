# ZKP-based-scholarship-verification-system
A rule-based neural network model evaluates scholarship eligibility using three numerical inputs: GPA, family income, and test score. Encoded logical thresholds (GPA ≥ 8.5, income ≤ ₹100,000, score ≥ 85) determine binary classification. Exported to ONNX for ZK-compatible inference verification.
Step-by-Step Workflow
1. Define & Export the Model
We use a simple PyTorch model implementing business rules:

python
Copy
Edit
approved = (gpa >= 8.5) & (income <= 100000) & (score >= 85)
Export to ONNX:

python
Copy
Edit
torch.onnx.export(model, example_input, "model.onnx", ...)
2. Create settings.json
json
Copy
Edit
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
input_visibility: whether inputs are public or private.

tolerance: acceptable precision (used for floating-point ops).

3. Compile the Model
Compile the ONNX model into a ZK circuit:

bash
Copy
Edit
ezkl compile-circuit -M model.onnx --settings-path settings.json
🔹 Output: model.compiled

4. Create the Input File
Example input:

json
Copy
Edit
{
  "input_data": [[9.2, 95000, 92]]
}
Save it as input.json.

5. Generate Witness
Compute intermediate values of model inference:

bash
Copy
Edit
ezkl gen-witness --data input.json --compiled-circuit model.compiled --output witness.json
🔹 Output: witness.json

6. Generate SRS (Trusted Setup)
bash
Copy
Edit
ezkl gen-srs --srs-path kzg.srs --logrows 17
kzg.srs: Needed for cryptographic operations like commitments.

7. Setup Keys
bash
Copy
Edit
ezkl setup --compiled-circuit model.compiled --vk-path vk.key --pk-path pk.key --srs-path kzg.srs
vk.key: For verifying proofs

pk.key: For generating proofs

8. Generate the ZK Proof
bash
Copy
Edit
ezkl prove --witness witness.json --compiled-circuit model.compiled \
--pk-path pk.key --proof-path proof.json --srs-path kzg.srs
🔹 Output: proof.json
This file proves the model ran correctly on your input—without revealing that input.

9. Verify the Proof (Optional)
bash
Copy
Edit
ezkl verify --proof-path proof.json --compiled-circuit model.compiled \
--vk-path vk.key --srs-path kzg.srs
If successful, this proves the decision (e.g. scholarship approved) is valid and trusted.

🔐 Use-Case Significance
Privacy: No need to reveal GPA, income, or score.

Verifiability: Third parties can verify eligibility.

Modularity: You can update rules without touching the ZK system.

📚 Dependencies
Rust + Cargo

EZKL v7.2.0

Python (for model creation)

PyTorch (for model export)

🤝 Contributing
Feel free to fork this repo and extend it with:

Real ML models

Private inputs

EVM integration

🧩 Future Work
EVM verifier generation

Integration with web frontend

Multi-party setup (MPC ceremony)
