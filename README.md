# Python-Project
This project is a desktop-based Billing System developed using Python Tkinter. It provides a simple and efficient way to generate customer bills, manage products, calculate totals, and store billing details.


#CODE
import tkinter as tk
from tkinter import ttk, messagebox, filedialog
from tkinter.font import Font

# === Data: Grocery items with prices ===
grocery_items = {
    "Grains": {
        "Rice": 30.25,
        "Wheat Flour": 20.80,
        "Bread": 20.50,
        "Pasta": 10.50
    },
    "Fruits": {
        "Apples": 80.50,
        "Oranges": 30.50,
        "Bananas": 20.50
    },
    "Vegetables": {
        "Potatoes": 10.50,
        "Onions": 20.75,
        "Tomatoes": 10.40,
        "Carrots": 10.90
    },
    "Dairy": {
        "Milk": 20.0,
        "Yogurt": 30.0,
        "Cheese": 40.0
    },
    "Protein meat": {
        "Eggs": 40.0,
        "Chicken": 120.50,
        "Fish": 60.50,
        "goat meat": 200.50
    },
    "Other": {
        "Spices": 30.0,
        "Oils": 35.50,
        "Condiments": 20.00,
        "Beverages": 10.50
    }
}

# === Color Palette ===
BG_COLOR = "#e6ffe6"  # Light green background
PRIMARY_COLOR = "#388e3c"  # Darker green primary
SECONDARY_COLOR = "#81c784" # Medium green secondary
ACCENT_COLOR = "#d32f2f"   # Still using red for accent
TEXT_COLOR = "#2e7d32"    # Dark green text
BUTTON_HOVER = "#4caf50"  # Green button hover
CARD_COLOR = "#f1f8e9"    # Light green card
DASHBOARD_BG = "#c8e6c9"  # Medium light green dashboard
  

cart = []

# === Main Functions ===
def update_items(event=None):
    selected_category = combo_category.get()
    items = list(grocery_items.get(selected_category, {}).keys())
    combo_item['values'] = items
    combo_item.set('')  # Clear item
    entry_price.delete(0, tk.END)

def fill_price(event=None):
    category = combo_category.get()
    item = combo_item.get()
    price = grocery_items.get(category, {}).get(item)
    if price:
        entry_price.delete(0, tk.END)
        entry_price.insert(0, f"{price:.2f}")

def add_to_cart():
    name = combo_item.get().strip()
    price = entry_price.get().strip()
    quantity = entry_quantity.get().strip()

    if not name or not price or not quantity:
        messagebox.showerror("Error", "Please fill all fields", parent=root)
        return

    try:
        price = float(price)
        quantity = int(quantity)
        if price <= 0 or quantity <= 0:
            raise ValueError
        total = price * quantity
        cart.append({'name': name, 'price': price, 'quantity': quantity, 'total': total})
        update_cart()
        clear_entries()
    except ValueError:
        messagebox.showerror("Error", "Enter valid numbers for price and quantity", parent=root)

def update_cart():
    cart_listbox.delete(0, tk.END)
    for idx, item in enumerate(cart, 1):
        cart_listbox.insert(tk.END, f"{idx:2}. {item['name'][:20]:20} x{item['quantity']:<4} @ ${item['price']:<6.2f} = ${item['total']:7.2f}")
    update_totals()

def update_totals():
    subtotal = sum(item['total'] for item in cart)
    lbl_subtotal.config(text=f"Subtotal: ${subtotal:.2f}")

def clear_entries():
    combo_category.set('')
    combo_item.set('')
    entry_price.delete(0, tk.END)
    entry_quantity.delete(0, tk.END)
    combo_category.focus()

def calculate_total():
    if not cart:
        messagebox.showwarning("Empty Cart", "Please add items to the cart before calculating.", parent=root)
        return

    try:
        gst = float(entry_gst.get()) / 100 if entry_gst.get() else 0
        discount = float(entry_discount.get()) if entry_discount.get() else 0
        if gst < 0 or discount < 0:
            raise ValueError
    except ValueError:
        messagebox.showerror("Invalid Input", "Enter valid positive numbers for GST and Discount.", parent=root)
        return

    subtotal = sum(item['total'] for item in cart)
    gst_amount = subtotal * gst
    total = subtotal + gst_amount - discount

    item_lines = "\n".join(
        f"{i+1:2}. {item['name'][:20]:20} x{item['quantity']:<4} @ ${item['price']:<6.2f} = ${item['total']:7.2f}"
        for i, item in enumerate(cart)
    )

    bill = f"""
====== FINAL BILL ======

Items:
{item_lines}

-----------------------
Subtotal:      ${subtotal:>8.2f}
GST ({gst*100:.0f}%):    ${gst_amount:>8.2f}
Discount:      ${discount:>8.2f}
-----------------------
TOTAL:        ${total:>8.2f}
"""
    messagebox.showinfo("Bill Summary", bill.strip(), parent=root)
    return bill

def remove_selected():
    selected_index = cart_listbox.curselection()
    if not selected_index:
        messagebox.showwarning("No Selection", "Please select an item to remove.", parent=root)
        return
    index = selected_index[0]
    del cart[index]
    update_cart()

def print_bill():
    if not cart:
        messagebox.showwarning("Empty Cart", "No items in cart to print.", parent=root)
        return

    bill_text = calculate_total()

    file_path = filedialog.asksaveasfilename(
        defaultextension=".txt",
        filetypes=[("Text Files", ".txt"), ("All Files", ".*")],
        title="Save Bill As",
        initialfile="Invoice.txt"
    )

    if file_path:
        try:
            with open(file_path, 'w') as file:
                file.write(bill_text)
            messagebox.showinfo("Success", f"Bill saved to:\n{file_path}", parent=root)
        except Exception as e:
            messagebox.showerror("Error", f"Failed to save bill:\n{str(e)}", parent=root)

# === GUI Setup ===
root = tk.Tk()
root.title("Retail Billing System")
root.geometry("850x700")
root.configure(bg=DASHBOARD_BG)  # Changed to dashboard background color

style = ttk.Style()
style.theme_use('clam')
style.configure('.', background=DASHBOARD_BG)  # Changed to dashboard background color
style.configure('TLabel', background=DASHBOARD_BG, foreground=TEXT_COLOR)  # Changed background
style.configure('TFrame', background=DASHBOARD_BG)  # Changed background
style.configure('TButton', background=PRIMARY_COLOR, foreground='white', padding=6, font=('Segoe UI', 10, 'bold'))
style.map('TButton', background=[('active', BUTTON_HOVER)])
style.configure('TLabelFrame', background=CARD_COLOR, padding=10)
style.configure('Accent.TButton', background=ACCENT_COLOR, foreground='white')
style.map('Accent.TButton', background=[('active', '#d33434')])

header_font = Font(family='Segoe UI', size=18, weight='bold')

# Main container frame with background color
main_container = ttk.Frame(root)
main_container.pack(fill="both", expand=True, padx=10, pady=10)
main_container.configure(style='TFrame')

# Header
header = ttk.Label(main_container, text="RETAIL BILLING SYSTEM", font=header_font, foreground=PRIMARY_COLOR)
header.pack(pady=20)

# Entry Frame
entry_frame = ttk.LabelFrame(main_container, text=" Add New Item ")
entry_frame.pack(fill="x", padx=20, pady=10)

fields_frame = ttk.Frame(entry_frame)
fields_frame.pack(fill="x")

# Category Dropdown
ttk.Label(fields_frame, text="Category:").grid(row=0, column=0, padx=5, pady=5, sticky="e")
combo_category = ttk.Combobox(fields_frame, values=list(grocery_items.keys()), state="readonly", width=18)
combo_category.grid(row=1, column=0, padx=5, pady=5)
combo_category.bind("<<ComboboxSelected>>", update_items)

# Item Dropdown
ttk.Label(fields_frame, text="Item:").grid(row=0, column=1, padx=5, pady=5, sticky="e")
combo_item = ttk.Combobox(fields_frame, state="readonly", width=20)
combo_item.grid(row=1, column=1, padx=5, pady=5)
combo_item.bind("<<ComboboxSelected>>", fill_price)

# Price Entry
ttk.Label(fields_frame, text="Price:").grid(row=0, column=2, padx=5, pady=5, sticky="e")
entry_price = ttk.Entry(fields_frame, width=10)
entry_price.grid(row=1, column=2, padx=5, pady=5)

# Quantity Entry
ttk.Label(fields_frame, text="Quantity:").grid(row=0, column=3, padx=5, pady=5, sticky="e")
entry_quantity = ttk.Entry(fields_frame, width=10)
entry_quantity.grid(row=1, column=3, padx=5, pady=5)

# Add Button
add_button = ttk.Button(fields_frame, text="➕ Add to Cart", command=add_to_cart)
add_button.grid(row=1, column=4, padx=10)

# Cart Frame
cart_frame = ttk.LabelFrame(main_container, text=" Shopping Cart ")
cart_frame.pack(fill="both", expand=True, padx=20, pady=10)

scrollbar = ttk.Scrollbar(cart_frame)
scrollbar.pack(side="right", fill="y")

cart_listbox = tk.Listbox(
    cart_frame, width=100, height=14, font=('Consolas', 10),
    bg="white", fg=TEXT_COLOR,
    selectbackground=ACCENT_COLOR, selectforeground="white",
    yscrollcommand=scrollbar.set, relief="flat"
)
cart_listbox.pack(fill="both", expand=True, padx=5, pady=5)
scrollbar.config(command=cart_listbox.yview)

# Remove Button
remove_button = ttk.Button(cart_frame, text="🗑️ Remove Selected", command=remove_selected)
remove_button.pack(anchor="w", padx=5, pady=5)

# Totals
totals_frame = ttk.Frame(main_container)
totals_frame.pack(fill="x", padx=20, pady=(0, 10))

lbl_subtotal = ttk.Label(totals_frame, text="Subtotal: $0.00", font=('Segoe UI', 10, 'bold'))
lbl_subtotal.pack(side="left")

tax_frame = ttk.Frame(totals_frame)
tax_frame.pack(side="right")

ttk.Label(tax_frame, text="GST %:").pack(side="left", padx=5)
entry_gst = ttk.Entry(tax_frame, width=6)
entry_gst.pack(side="left")

ttk.Label(tax_frame, text="Discount $:").pack(side="left", padx=5)
entry_discount = ttk.Entry(tax_frame, width=8)
entry_discount.pack(side="left")

# Bottom Buttons
bottom_frame = ttk.Frame(main_container)
bottom_frame.pack(fill="x", padx=20, pady=10)

ttk.Button(bottom_frame, text="🖨️ Print/Save Bill", command=print_bill).pack(side="left")
ttk.Button(bottom_frame, text="💳 Generate Bill", style='Accent.TButton', command=calculate_total).pack(side="right")

combo_category.focus()
root.mainloop()

