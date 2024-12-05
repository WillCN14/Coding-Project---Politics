import csv

# Classes
class MP:
    def __init__(self, firstname, surname, party, constituency):
        self.firstname = firstname
        self.surname = surname
        self.party = party
        self.constituency = constituency

    def politician(self):
        return f"{self.firstname} {self.surname}"

    def perPolitics(self):
        return f"{self.party}, {self.constituency}"

    def perCandidate(self):
        return f"{self.politician()} - {self.perPolitics()}"


class Party:
    def __init__(self, name):
        self.name = name
        self.total_votes = 0

    def add_votes(self, votes):
        self.total_votes += votes


class Constituency:
    def __init__(self, name, county, region, result):
        self.name = name
        self.county = county
        self.region = region
        self.result = result
        self.party_votes = {}

    def add_votes(self, party_name, votes):
        if party_name not in self.party_votes:
            self.party_votes[party_name] = 0
        self.party_votes[party_name] += votes


# Helper Functions
def menu():
    print("""
Welcome! Please choose from one of the following options:
1) All winning candidates
2) All participating parties
3) Parties' final stats
4) Constituency details
5) Exit
""")


def load_data(file_name):
    constituencies = {}
    parties = {}
    politicians = {}

    try:
        with open(file_name, 'r', newline="") as csv_file:
            csv_reader = csv.DictReader(csv_file)

            for row in csv_reader:
                # Constituency data
                constituency_name = row["Constituency name"]
                if constituency_name not in constituencies:
                    constituencies[constituency_name] = Constituency(
                        name=constituency_name,
                        county=row["Country name"],
                        region=row["Region name"],
                        result=row["Result"]
                    )
                constituency = constituencies[constituency_name]

                # Party votes
                parties_list = ['Con', 'Lab', 'LD', 'RUK', 'Green', 'SNP', 'PC', 'SF', 'SDLP', 'UUP', 'APNI', 'All other candidates']
                for party_name in parties_list:
                    votes = int(row.get(party_name, 0))
                    constituency.add_votes(party_name, votes)
                    if party_name not in parties:
                        parties[party_name] = Party(party_name)
                    parties[party_name].add_votes(votes)

                # Politician data
                if row["Member first name"] and row["Member surname"]:
                    politician_name = f"{row['Member first name']} {row['Member surname']}"
                    if politician_name not in politicians:
                        politicians[politician_name] = MP(
                            firstname=row["Member first name"],
                            surname=row["Member surname"],
                            party=row["First party"],
                            constituency=constituency_name
                        )
                    constituency.result = politician_name

    except FileNotFoundError:
        print("Error: File not found. Please check the file name and path.")
    except Exception as e:
        print(f"An error occurred: {e}")

    return politicians, parties, constituencies


def display_winning_candidates(politicians):
    for name, politician in politicians.items():
        print(politician.perCandidate())


def display_participating_parties(parties):
    print("Participating Parties:")
    for party in parties.keys():
        print(party)


def display_party_final_stats(parties):
    total_votes = sum(party.total_votes for party in parties.values())
    print("Party Final Stats:")
    for party in parties.values():
        percentage = (party.total_votes / total_votes) * 100
        print(f"{party.name}: {party.total_votes} votes ({percentage:.2f}%)")


def display_constituency_details(constituencies):
    constituency_name = input("Enter constituency name: ").strip()
    constituency = constituencies.get(constituency_name)
    if constituency:
        print(f"Constituency: {constituency.name}")
        print(f"Result: {constituency.result}")
        print("Votes by Party:")
        for party, votes in constituency.party_votes.items():
            print(f"  {party}: {votes}")
    else:
        print("Constituency not found.")


# Main Program
def main():
    file_name = 'C:\\Users\\crowd\\OneDrive\\Documents\\EditedData.CSV'
    politicians, parties, constituencies = load_data(file_name)

    while True:
        menu()
        choice = input("Enter your choice (1-5): ").strip()
        if choice == "1":
            display_winning_candidates(politicians)
        elif choice == "2":
            display_participating_parties(parties)
        elif choice == "3":
            display_party_final_stats(parties)
        elif choice == "4":
            display_constituency_details(constituencies)
        elif choice == "5":
            print("Exiting program. Goodbye!")
            break
        else:
            print("Invalid choice. Please try again.")


# Run the program
if __name__ == "__main__":
    main()
