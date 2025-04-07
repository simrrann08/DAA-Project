import tkinter as tk
import sqlite3
import datetime

# ---------------- Database Setup ----------------
conn = sqlite3.connect("cafe.db")
cursor = conn.cursor()
cursor.execute('''
    CREATE TABLE IF NOT EXISTS receipts (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        item TEXT,
        quantity INTEGER,
        cost INTEGER,
        total INTEGER,
        timestamp TEXT
    )
''')
conn.commit()

# ---------------- Data Structures ----------------
menu_items = [
    {"name": "Coffee", "price": 50},
    {"name": "Tea", "price": 30},
    {"name": "Sandwich", "price": 80},
    {"name": "Burger", "price": 100},
    {"name": "Pasta", "price": 120},
    {"name": "Cake", "price": 60},
]

def linear_search(name):
    for item in menu_items:
        if item["name"].lower() == name.lower():
            return item
    return None

class Stack:
    def __init__(self):
        self.stack = []
    def push(self, val):
        self.stack.append(val)
    def pop(self):
        return self.stack.pop() if self.stack else None

# ---------------- Main App ----------------
class CafeBilling:
    def __init__(self, root):
        self.root = root
        self.root.title("Café Management")
        self.root.configure(bg="#f4e3c1")
        self.stack = Stack()

        tk.Label(root, text="Café Menu", font=("Arial", 16, "bold"), bg="#f4e3c1").grid(row=0, column=0, columnspan=2, pady=10)

        # Headings
        tk.Label(root, text="Item", font=("Arial", 11, "bold"), bg="#f4e3c1").grid(row=1, column=0, padx=30, sticky='w')
        tk.Label(root, text="Quantity", font=("Arial", 11, "bold"), bg="#f4e3c1").grid(row=1, column=1)

        self.entries = {}
        for i, item in enumerate(menu_items):
            tk.Label(root, text=f"{item['name']} (₹{item['price']})", bg="#f4e3c1", font=("Arial", 10)).grid(row=i+2, column=0, sticky='w', padx=30)
            qty_entry = tk.Entry(root, width=5)
            qty_entry.grid(row=i+2, column=1)
            self.entries[item['name']] = qty_entry

        # Button to calculate
        tk.Button(root, text="Calculate & Show Receipt", command=self.calculate, bg="lightgrey").grid(row=len(menu_items)+2, column=0, columnspan=2, pady=10)

        # Total Label
        self.total_label = tk.Label(root, text="Total: ₹0", font=("Arial", 12, "bold"), bg="#f4e3c1")
        self.total_label.grid(row=len(menu_items)+3, column=0, columnspan=2, pady=5)

        # Receipt Box
        self.receipt = tk.Text(root, height=12, width=40, font=("Courier", 10))
        self.receipt.grid(row=len(menu_items)+4, column=0, columnspan=2, pady=10, padx=30)

    def calculate(self):
        self.receipt.delete("1.0", tk.END)
        total = 0
        lines = []
        timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")

        for name, entry in self.entries.items():
            try:
                qty = int(entry.get())
            except:
                qty = 0
            item = linear_search(name)
            if item and qty > 0:
                cost = qty * item['price']
                lines.append(f"{name:<10} x {qty:<2} = ₹{cost}")
                total += cost

                # Save to database
                cursor.execute("INSERT INTO receipts (item, quantity, cost, total, timestamp) VALUES (?, ?, ?, ?, ?)",
                               (name, qty, cost, total, timestamp))
                conn.commit()

        for line in lines:
            self.receipt.insert(tk.END, line + "\n")

        self.receipt.insert(tk.END, "------------------------\n")
        self.receipt.insert(tk.END, f"Total: ₹{total}")
        self.total_label.config(text=f"Total: ₹{total}")
        self.stack.push((lines, total))

# ---------------- Run App ----------------
if __name__ == "__main__":
    root = tk.Tk()
    app = CafeBilling(root)
    root.mainloop()
