# SIBMS Final User Manual

## Overview
SIBMS is a sales, inventory, and bookkeeping management system for a retail hardware business. The system supports the recording of sales, updating inventory balances, and keeping a consistent transaction record for stock and accounting activity.

## Intended Users

- Sales Attendant
- Manager / Shop Owner
- System Administrator

## Main Features

- Login and user authentication
- Product catalog browsing
- Sales creation workflow
- Inventory quantity reduction on successful sale
- Payment method selection
- Receipt record generation
- Stock and ledger consistency checks

## How to Use the System

### 1. Login
Use a valid user account provided by the administrator. Authentication is required before access to the sales workflow.

### 2. Browse Products
Open the product catalog to review available items and current stock levels.

### 3. Create a Sale
- Select the product items required by the customer
- Enter quantity values
- Choose a payment method
- Review the total
- Submit the transaction

### 4. Confirmation
On successful submission:
- the sale is saved
- stock is reduced
- a receipt number is created
- inventory and ledger entries remain synchronized

### 5. Error Handling
If the requested quantity exceeds stock availability, the system should reject the transaction with a stock conflict response and prevent partial updates.

## Known Limitations

This user manual should be updated with the actual screenshots and operation flow of the final deployed system once the live application is reviewed and confirmed by the project team.
