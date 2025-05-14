Hi Team,

Sharing a quick update on recent enhancements:
	•	Leaver Process:
The previous script was non-functional due to missing components. I’ve developed a new shell script, automated it via OEM, and added a feature to track and report locked users. After execution, an email is sent with the list of locked users along with their corresponding database names.
	•	Schema Refresh:
The schema refresh process is now fully automated using a config-driven shell script, eliminating the need for manual intervention. Previously, we had to log in during weekends to take export backups and perform refreshes. Now, the entire flow—including export, cleanup, and import—is automated and scheduled via OEM, making the refresh process faster, more accurate, and completely hands-free. A completion report is sent via email after each run.
	•	Documentation:
SOPs for both processes have been created and uploaded to OneDrive for reference.

Feel free to review and share any feedback.

Thanks & Regards,
[Your Name]
