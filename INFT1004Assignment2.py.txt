
#Author-1:Bhargav Patadia c3440219
#INFT1004 - Introduction to Programming-cafe inventory management - part 2


import matplotlib.pyplot as plt
import pandas as pd
import os

# Predefined maximum values for known items
MAX_VALUES = {
    'coffee': 2,
    'tea': 2,
    'milk': 5,
    'chocolate cake': 10
}


def initialise_inventory(filename):
    """Initialize the inventory from an external file."""
    try:
        inventory_df = pd.read_csv(filename, delimiter=':', header=None, names=['item', 'amount'])
        inventory_df['item'] = inventory_df['item'].str.strip().str.lower()
        inventory_df['amount'] = inventory_df['amount'].astype(float)
        inventory = inventory_df.set_index('item')['amount'].to_dict()
    except FileNotFoundError:
        print("Inventory file is not found. Program will be quit.")
        exit()
    except ValueError:
        print("The File format is not right. Ensure it follows 'item:amount' format.")
        exit()
    return inventory


def add_product(inventory):
    """Add a new product in the inventory."""
    while True:
        product = input("Enter the name of the new product: ").strip().lower()
        if product in inventory:
            print("Product already exists. Please enter a different product name.")
        else:
            break
    while True:
        try:
            amount = float(input(f"Enter the initial inventory level for {product}: "))
            if amount >= 0:
                inventory[product] = amount
                MAX_VALUES[product] = 10  # Assume default max value for new items
                print(f"Product {product} added with the inventory level {amount}.")
                break
            else:
                print("Amount should be a positive number. Please try again.")
        except ValueError:
            print("Invalid input. Please enter a numeric value.")
    return inventory


def order_drink(inventory, drink):
    """Order a drink and update the inventory."""
    drink = drink.lower()
    if drink == "coffee":
        if inventory.get('coffee', 0) >= 0.02:
            inventory['coffee'] -= 0.02
            print("Coffee is ordered. Inventory updated.")
        else:
            print("Not enough coffee in inventory.")
    elif drink == "tea":
        if inventory.get('tea', 0) >= 0.02:
            inventory['tea'] -= 0.02
            print("Tea is ordered. Inventory updated.")
        else:
            print("Not enough tea in the inventory.")
    elif drink == "latte":
        if inventory.get('coffee', 0) >= 0.02 and inventory.get('milk', 0) >= 0.2:
            inventory['coffee'] -= 0.02
            inventory['milk'] -= 0.2
            print("Latte is ordered. Inventory updated.")
        else:
            print("There is Not enough coffee or milk in the inventory.")
    else:
        print("Option for the drink is invalid.")
    return inventory


def order_cake(inventory, product, pieces):
    """Order cake/pie and update the inventory."""
    product = product.lower()
    if product in inventory and inventory[product] >= pieces:
        inventory[product] -= pieces
        print(f"{pieces} pieces of {product} ordered. Inventory updated.")
    else:
        print(f"Not enough {product} in inventory or invalid product.")
    return inventory


def restock_inventory(inventory):
    """Restock the inventory based on user input."""
    for item in inventory.keys():
        while True:
            try:
                restock_amount = float(input(f"Enter amount of {item} to add: "))
                max_value = MAX_VALUES.get(item, 10)  # Default max value for new items
                if 0 <= restock_amount + inventory[item] <= max_value:
                    inventory[item] += restock_amount
                    print(f"{item} restocked. New amount: {inventory[item]}")
                    break
                else:
                    print(f"Restock amount exceeds maximum allowed ({max_value}). Please try again.")
            except ValueError:
                print("Invalid input. Please enter a numeric value.")
    return inventory


def inventory_analysis(inventory):
    """Analyze the inventory and provide a report."""
    for item, amount in inventory.items():
        max_value = MAX_VALUES.get(item, 10)  # Default max value for new items
        if amount >= 0.9 * max_value:
            print(f"Inventory high for {item}")
        elif 0.5 * max_value <= amount < 0.9 * max_value:
            print(f"Inventory ok for {item}")
        else:
            print(f"Low inventory for {item}")

    # Plotting the inventory levels in the graph
    items = list(inventory.keys())
    levels = list(inventory.values())
    plt.bar(items, levels)
    plt.xlabel('Items')
    plt.ylabel('Inventory Levels')
    plt.title('Cafe Inventory Levels')
    plt.show()


def quantity_of_drinks(inventory, drink):
    """Calculate the quantity of drinks that can be made with the current inventory."""
    drink = drink.lower()
    if drink == "coffee":
        print(f"Number of the coffee drinks that can be made: {int(inventory['coffee'] // 0.02)}")
    elif drink == "tea":
        print(f"Number of the tea drinks that can be made: {int(inventory['tea'] // 0.02)}")
    elif drink == "latte":
        print(
            f"Number of the latte drinks that can be made: {int(min(inventory['coffee'] // 0.02, inventory['milk'] // 0.2))}")
    else:
        print("Drink option is invalid.")


def main():
    filename = 'CafeInventory.txt'
    if not os.path.isfile(filename):
        print(f"Inventory file {filename} not found. Program will quit.")
        return

    inventory = initialise_inventory(filename)

    while True:
        print("\nMenu:")
        print("1. Add a new product")
        print("2. Order one drink")
        print("3. Order cake/pie")
        print("4. Inventory analysis")
        print("5. Restock the inventory")
        print("6. Quantity of drinks available")
        print("7. Quit")

        choice = input("Choose an action: ").strip()

        if choice == "1":
            inventory = add_product(inventory)
        elif choice == "2":
            drink = input("Enter drink (coffee, tea, latte): ").strip()
            inventory = order_drink(inventory, drink)
        elif choice == "3":
            print("Current cakes and pies in inventory:")
            for item, amount in inventory.items():
                if item not in ['coffee', 'tea', 'milk']:
                    print(f"{item}: {amount} pieces")
            product = input("Enter the name of the product (cake or pie): ").strip()
            pieces = int(input("Enter the number of pieces: ").strip())
            inventory = order_cake(inventory, product, pieces)
        elif choice == "4":
            inventory_analysis(inventory)
        elif choice == "5":
            inventory = restock_inventory(inventory)
        elif choice == "6":
            drink = input("Enter drink (coffee, tea, latte): ").strip()
            quantity_of_drinks(inventory, drink)
        elif choice == "7":
            print("Program gets quit.")
            break
        else:
            print("The choice is invalid. Please try again.")


if __name__ == "__main__":
    main()

