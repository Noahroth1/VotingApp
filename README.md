# Voting App(Python + SQLite)
A command-line voting application built with Python and SQLite that demonstrates database-backed CRUD operations, admin authorization, and persistent vote tracking.

This project was created to practice object-oriented programming, SQL integration, and menu-driven application design.

class Voting:
    def __init__(self, db_name="Voting.db"):
        self.conn = sqlite3.connect(db_name)
        self.conn.execute("PRAGMA foreign_keys = ON;")
        self.admin_password = "Noah123"

        self.create_candidate_table()
        self.create_user_table()

    def create_user_table(self):
        query = """
        CREATE TABLE IF NOT EXISTS voters (
            user_id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_name TEXT NOT NULL UNIQUE
        );
        """
        self.conn.execute(query)
        self.conn.commit()

    def create_candidate_table(self):
        query = """
        CREATE TABLE IF NOT EXISTS candidates (
            candidate_id INTEGER PRIMARY KEY AUTOINCREMENT,
            candidate_name TEXT NOT NULL UNIQUE,
            candidate_votes INTEGER NOT NULL DEFAULT 0
        );
        """
        self.conn.execute(query)
        self.conn.commit()

    def show_candidates(self):
        cursor = self.conn.execute(
            "SELECT candidate_name, candidate_votes FROM candidates ORDER BY candidate_name;"
        )
        rows = cursor.fetchall()
        if not rows:
            print("No candidates yet. Ask admin to add candidates first.")
            return False

        print("\nCandidates:")
        for name, votes in rows:
            print(f"- {name}: {votes}")
        print()
        return True

    def add_candidate(self):
        name = input("Candidate name: ").strip()
        if not name:
            print("Name can't be empty.")
            return

        try:
            self.conn.execute(
                "INSERT INTO candidates(candidate_name, candidate_votes) VALUES(?, 0);",
                (name,),
            )
            self.conn.commit()
            print(f"Added candidate: {name}")
        except sqlite3.IntegrityError:
            print("That candidate already exists.")

    def remove_candidate(self):
        name = input("Candidate name to remove: ").strip()
        if not name:
            print("Name can't be empty.")
            return

        cur = self.conn.execute(
            "DELETE FROM candidates WHERE candidate_name = ?;",
            (name,),
        )
        self.conn.commit()

        if cur.rowcount == 0:
            print(f"{name} is not a candidate.")
        else:
            print(f"{name} removed.")

    def cast_vote(self):
        if not self.show_candidates():
            return

        voter_name = input("What is your name? ").strip()
        if not voter_name:
            print("Name can't be empty.")
            return
        try:
            self.conn.execute(
                "INSERT INTO voters(user_name) VALUES(?);",
                (voter_name,),
            )
            self.conn.commit()
        except sqlite3.IntegrityError:
            print("Sorry — you already voted (name already recorded).")
            return

        choice = input("Who would you like to vote for? ").strip()
        if not choice:
            print("Vote cancelled.")
            return

        cur = self.conn.execute(
            "UPDATE candidates SET candidate_votes = candidate_votes + 1 WHERE candidate_name = ?;",
            (choice,),
        )
        self.conn.commit()

        if cur.rowcount == 0:
            print(f"Sorry — '{choice}' is not a candidate. (Your name was recorded as having voted.)")
            print("Admin could delete your name from voters if you want to allow a retry.")
        else:
            print("Vote added ")

    def show_result(self):
        cursor = self.conn.execute(
            "SELECT candidate_name, candidate_votes FROM candidates ORDER BY candidate_votes DESC, candidate_name ASC;"
        )
        rows = cursor.fetchall()

        if not rows:
            print("No candidates found.")
            return

        print("\nFinal Results:")
        for name, votes in rows:
            print(f"- {name}: {votes}")

        winner_name, winner_votes = rows[0]
        print(f"\nWinner: {winner_name} with {winner_votes} votes\n")

    def admin_login(self):
        attempts_left = 3
        while attempts_left > 0:
            password = input("Admin password: ")
            if password == self.admin_password:
                print("Access granted.")
                self.admin_menu()
                return
            attempts_left -= 1
            print(f"Access denied. Attempts left: {attempts_left}")
        print("Too many failed attempts.")

    def admin_menu(self):
        while True:
            choice = input("\nAdmin: 1 add | 2 remove | 3 view results | 4 back\n> ").strip()
            if choice == "1":
                self.add_candidate()
            elif choice == "2":
                self.remove_candidate()
            elif choice == "3":
                self.show_result()
            elif choice == "4":
                print("Leaving admin menu.")
                return
            else:
                print("Invalid option.")

    def main_menu(self):
        print("Voting app menu:")
        while True:
            choice = input("\n1 admin | 2 vote | 3 results | 4 exit\n> ").strip()
            if choice == "1":
                self.admin_login()
            elif choice == "2":
                self.cast_vote()
            elif choice == "3":
                self.show_result()
            elif choice == "4":
                print("Thank you, goodbye!")
                self.conn.close()
                break
            else:
                print("Invalid option.")


if __name__ == "__main__":
    Voting().main_menu()
