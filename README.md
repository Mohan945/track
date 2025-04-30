•	During the refresh, the product (application) reads production data that includes user creation instructions.
	•	Those users get auto-created before your import step.
	•	When the import step tries to re-import those users/accounts, it fails because the user/schema already exists.
	•	Passwords and other attributes are not re-applied since the user creation step is skipped.
