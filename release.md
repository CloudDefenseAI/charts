Release Date : 29.05.2024
Release Notes : 4.8.8

New feature list:
1) Delta Scan
2) Application team assignment: We have now provided application team assignment mechanism based on team pseudonym.
3) Automatic Jira ticket creation: We have provided automatic Jira ticket creation for the new detected vulnerabilities, configurable to enable/disable at severities level.
4) Recommendation/Solution under vulnerability report: We have provided recommendations under the CTO report.
5) Multi select False Positive and Allowed list: We have added ability to select multiple vulnerabilities to mark as false positive and add to allowed list.
6) DevSecOps configure with OKTA: We have provided SSO integration with OKTA. 
7) Jira ticket having alert back to the vulnerability and show latest status: We have added a link back under the Jira ticket to have link back to the vulnerability for which it was created.
8) Show "Age" of a vulnerability: We have added age detail, day wise to show the age of a vulnerability when it was first detected under CTO report.
9) User to be able to request to add directories for exclusion: Added new interface that allows admins to manage your teammate's requests for excluding file-paths.

Improvements:
1) Application Page Refactor: We have refactored our Application page, adding branches and pull requests view on the expansion of the application from list. By clicking on the link icon next to the application name, users can navigate directly to the specific source. Additionally, users can now filter to view only branches or pull requests by using the buttons located below the application name.
2) Fixed GitHub group name not coming for the repo name.
3) Scan steps handling asynchronously, removing the bottle-neck for running multiple scans concurrently.
4) Fixed list of UI issues.