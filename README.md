# hackathon
import json
from collections import defaultdict

class QuickSplit:
    def __init__(self):
        self.expenses = []
        self.people = set()
    
    def add_expense(self, payer, amount, participants, split_type='equal', custom_shares=None):
        """
        Add a new expense to the tracker
        
        Args:
            payer: Who paid for this expense
            amount: Total amount paid
            participants: List of people who share this expense
            split_type: 'equal' or 'custom'
            custom_shares: Required if split_type is 'custom' (dict of {person: share})
        """
        if split_type == 'equal':
            share = amount / len(participants)
            shares = {p: share for p in participants}
        elif split_type == 'custom':
            if not custom_shares or sum(custom_shares.values()) != amount:
                raise ValueError("Invalid custom shares")
            shares = custom_shares
        else:
            raise ValueError("Invalid split type")
            
        self.expenses.append({
            'payer': payer,
            'amount': amount,
            'shares': shares
        })
        self.people.update(participants)
        self.people.add(payer)
    
    def calculate_balances(self):
        """Calculate net balance for each person"""
        balances = defaultdict(float)
        
        for expense in self.expenses:
            payer = expense['payer']
            shares = expense['shares']
            
            balances[payer] += expense['amount']
            for person, share in shares.items():
                balances[person] -= share
                
        return balances
    
    def simplify_settlements(self, balances):
        """Simplify debts using graph theory to minimize transactions"""
        creditors = []
        debtors = []
        
        for person, balance in balances.items():
            if balance > 0:
                creditors.append((person, balance))
            elif balance < 0:
                debtors.append((person, -balance))
                
        creditors.sort(key=lambda x: x[1], reverse=True)
        debtors.sort(key=lambda x: x[1], reverse=True)
        
        settlements = []
        i = j = 0
        
        while i < len(creditors) and j < len(debtors):
            creditor, credit_amt = creditors[i]
            debtor, debt_amt = debtors[j]
            
            settlement_amt = min(credit_amt, debt_amt)
            settlements.append((debtor, creditor, settlement_amt))
            
            creditors[i] = (creditor, credit_amt - settlement_amt)
            debtors[j] = (debtor, debt_amt - settlement_amt)
            
            if creditors[i][1] == 0:
                i += 1
            if debtors[j][1] == 0:
                j += 1
                
        return settlements
    
    def generate_report(self, settlements):
        """Generate a human-readable settlement report"""
        if not settlements:
            return "No settlements needed - all balances are even."
            
        report = ["\nSettlement Instructions:", "="*25]
        for debtor, creditor, amount in settlements:
            report.append(f"{debtor} should pay {creditor} ${amount:.2f}")
        
        return "\n".join(report)
    
    def save_to_file(self, filename, data):
        """Save data to a JSON file"""
        with open(filename, 'w') as f:
            json.dump(data, f, indent=2)
    
    def load_from_file(self, filename):
        """Load data from a JSON file"""
        with open(filename, 'r') as f:
            data = json.load(f)
            self.expenses = data.get('expenses', [])
            self.people = set(data.get('people', []))
    
    def run_cli(self):
        """Command-line interface for QuickSplit"""
        print("QuickSplit - Expense Tracking & Splitting Tool")
        print("="*45)
        
        while True:
            print("\nMenu:")
            print("1. Add new expense")
            print("2. Show settlements")
            print("3. Save to file")
            print("4. Load from file")
            print("5. Exit")
            
            choice = input("Enter your choice: ")
            
            if choice == '1':
                payer = input("Who paid for this expense? ")
                amount = float(input("Amount paid: "))
                participants = input("Participants (comma separated): ").split(',')
                participants = [p.strip() for p in participants]
                
                split_type = input("Split type (equal/custom): ").lower()
                custom_shares = None
                
                if split_type == 'custom':
                    custom_shares = {}
                    print("Enter each person's share:")
                    for p in participants:
                        share = float(input(f"{p}'s share: "))
                        custom_shares[p] = share
                
                self.add_expense(payer, amount, participants, split_type, custom_shares)
                print("Expense added successfully!")
            
            elif choice == '2':
                balances = self.calculate_balances()
                settlements = self.simplify_settlements(balances)
                print(self.generate_report(settlements))
            
            elif choice == '3':
                filename = input("Enter filename to save: ")
                data = {
                    'expenses': self.expenses,
                    'people': list(self.people)
                }
                self.save_to_file(filename, data)
                print(f"Data saved to {filename}")
            
            elif choice == '4':
                filename = input("Enter filename to load: ")
                self.load_from_file(filename)
                print(f"Data loaded from {filename}")
            
            elif choice == '5':
                print("Exiting QuickSplit. Goodbye!")
                break
            
            else:
                print("Invalid choice. Please try again.")

if __name__ == "__main__":
    app = QuickSplit()
    app.run_cli()
