# Using the Leapp JSON Report
The role leverages the `/var/log/leapp/leapp-report.json` file to manage the upgrade lifecycle through three main actions:

* Generation: The `leapp preupgrade` task creates the JSON report on the target RHEL 9 system.

* Extraction: The `ansible.builtin.fetch` module pulls the report back to the control node (your local machine) for archiving and manual review.

* Validation: The role inspects the results. If the report indicates any inhibitors (critical blockers), the playbook is designed to fail before the leapp upgrade command is even attempted.

This "Report-First" approach ensures that the system is never placed in a state of high risk without first meeting all technical requirements.