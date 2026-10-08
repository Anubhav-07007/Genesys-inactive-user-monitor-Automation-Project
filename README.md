# Genesys Cloud Inactive User Monitoring Automation

Automatically find inactive users in Genesys Cloud and generate an Excel report for the administrator to review, with no manual checking.

## The problem

Over time, Genesys Cloud accounts pile up for people who no longer log in. Checking each user by hand is slow, and deleting accounts without a review is risky. The administrator needed a clear list containing:

- User name
- Email address
- User ID
- Department
- Last login date
- Inactive days
- Current state

The Genesys Cloud UI does not give this view with inactivity categories, so a report had to be built from the API data.

## The solution

1. **Prototyped in Postman.** Tested the Users API to get active users along with their last login date, and checked the inactivity logic with a small script.
2. **Automated in Python.** Built a script that:
   - Authenticates with Genesys Cloud using OAuth (Client Credentials)
   - Fetches all active users with pagination
   - Calculates how many days each user has been inactive
   - Sorts users into inactivity categories
   - Flags users who have never logged in
   - Writes everything to an Excel report with a timestamped filename
3. **Kept a human in the loop.** The report goes to the Genesys administrator for review and approval. The automation never deletes or changes any account.

## Inactivity categories

| Days since last login | Category |
|---|---|
| 30-59 | Warning |
| 60-89 | Review |
| 90 or more | Candidate for Removal |
| No login recorded | Never Logged In |

Users inactive for fewer than 30 days are not included in the report.

## Workflow

```
OAuth token -> Users API (paginated) -> Calculate inactive days
                                   |
                     Categorise users (Warning / Review / Removal)
                                   |
                  Excel report (timestamped filename) -> Administrator review
```

## Tech stack

- Genesys Cloud Platform API (OAuth Client Credentials)
- Python 3 (requests, pandas, openpyxl, python-dotenv)
- Postman (API testing)
- VS Code

## Setup

1. Clone the repo, create a virtual environment, and install the packages listed in `requirements.txt`.
2. Create an OAuth client in Genesys Cloud (Client Credentials grant) with only the read permission needed to view users.
3. Copy `.env.example` to `.env` and fill in your region, client ID, and client secret.
4. Run the main script. The Excel report is saved in the `reports` folder.

## Security

- Credentials live only in `.env`, which is excluded by `.gitignore`.
- Use a dedicated OAuth client with least-privilege, read-only access.
- Never commit reports, because they contain user email addresses.
- Rotate the client secret if it is ever exposed.

## Project structure

```
.
├── inactive_users_report.py
├── requirements.txt
├── .env.example
├── .gitignore
├── postman/
├── tests/
├── reports/
└── README.md
```

## Lessons learned

- Postman is great for testing an API and checking the logic quickly, but Python is better for repeatable reports.
- Pagination is essential, otherwise users are silently missed in larger organisations.
- Users who have never logged in have no last login date, so they need separate handling.
- A report for admin approval is safer than automatic deletion.

## Possible improvements

- Run the report automatically every month
- Email the report to the administrator
- Exclude service and integration accounts
- Logging and retry handling

## Disclaimer

This is a personal learning project. All data shown is sample or anonymized. It is not affiliated with or endorsed by Genesys.
