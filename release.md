Release Date : 06.03.2024
Release Notes : 4.8.6

New feature list:
1) Import Logs Functionality on Organizational Level: We have made the import logs to show information for all Organization if `all teams` selector is enabled.
2) New Field Addition on Application Page: We have included the last scan of the current branch and the last scan in the app with the branch name under the Application page.
3) CTO Report TeamName: Added team name to CTO report.
4) CTO Report False Positive: Added False positive column under CTO report.
5) Time Series Graph: Updated time series graph with below 3 legends on daily basis, weekly basis, monthly basis, quarterly basis
    - Open Vulnerability trend
    - New  Vulnerability Identified trend
    - Closed/Fixed  Vulnerability 
6) Generate report for 30 days and 60 days
7) Updates Team admin acess role: Team admin can now invite new user/existing team member to any team.

Improvements:
1) Fixed Audit Log Search By date Filter: We have resolved audit log search on date filter.
2) Fixed the issue with parallel execution of scans: Fixed the CloudDefense processes hang issues with parallel execution of scans.
3) Fixed Ruby still has false positives: Fixed false positive in the Ruby Gemfile.lock.