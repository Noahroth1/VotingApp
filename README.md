# Voting App

A command-line voting application built with Python and SQLite. Administrators can manage candidates, voters can cast one vote each, and results are stored locally between sessions.

## Features

- Add and remove candidates through an administrator menu
- Limit voting to one vote per voter name
- Display candidates and their current vote totals
- Rank results by vote count and announce the winner
- Store candidates, voters, and results in a local SQLite database
- Reject empty names and duplicate candidates
- Limit administrator login to three attempts

## Requirements

- Python 3.6 or later
- SQLite support, included with standard Python installations
- No third-party packages

## Run the App

Clone the repository and move into the project directory:

```bash
git clone https://github.com/Noahroth1/VotingApp.git
cd VotingApp
```

Start the application:

```bash
python3 VotingApp
```

The main menu provides four options:

1. Open the administrator login
2. Cast a vote
3. View election results
4. Exit the application

## Administrator Menu

Select the administrator option from the main menu and enter the password configured in the `Voting` class. After logging in, an administrator can add candidates, remove candidates, or view results.

> **Note:** The current project uses a hard-coded password for demonstration purposes. A production application should store a securely hashed password outside the source code.

## How Voting Works

1. The application displays the available candidates.
2. The voter enters their name.
3. The voter chooses a candidate by entering the candidate's name.
4. The vote total is updated in the SQLite database.

Voter names must be unique, which provides a simple one-vote-per-name rule. This is suitable for a learning project, but it is not secure identity verification for a real election.

## Data Storage

The application creates `Voting.db` in the current directory when it first runs. It contains two tables:

| Table | Purpose |
| --- | --- |
| `candidates` | Stores candidate names and vote totals |
| `voters` | Stores voter names to prevent duplicate voting |

To start with a completely new local election, stop the application and remove `Voting.db`. This permanently deletes the locally stored candidates, voters, and results.

## Project Structure

```text
VotingApp/
├── VotingApp  # Python command-line application
└── README.md  # Project documentation
```
