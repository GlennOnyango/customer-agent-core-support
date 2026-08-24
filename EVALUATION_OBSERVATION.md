# Evaluation Observation

The Amazon Bedrock evaluation produced a **Correctness score of 1.00 across 9 prompts**. This suggests that the chatbot's responses closely matched the expected answers for the tested bug-report, FAQ, and unsupported-request scenarios. In particular, it generally routed requests correctly, supplied the expected support guidance, and avoided creating bug reports when they were not appropriate.

The perfect score should still be interpreted cautiously because the evaluation set is small. One response also stated that gift wrapping is not offered even though the reference answer only says that the FAQ does not provide this information; this claim should be removed to avoid inventing policy. As a next step, test a larger set of ambiguous, incomplete, and adversarial requests, verify that internal `<thinking>` text is never shown to users, and refine the unsupported-request prompt so the chatbot redirects to human support without making unverified claims.
