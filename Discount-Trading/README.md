# Dynamic Bulk Purchase Marketplace

## Overview

The Dynamic Bulk Purchase Marketplace is an advanced decentralized marketplace smart contract built on the Stacks blockchain. It enables merchants to establish sophisticated volume-based pricing strategies with automated discount calculations, comprehensive inventory management, secure payment processing, and multi-item bulk ordering capabilities.

## Features

### Core Functionality
- **Product Management**: Register, update, and manage product listings with inventory tracking
- **Volume-Based Pricing**: Configure flexible discount tiers to incentivize bulk purchases
- **Bulk Order Processing**: Support for multi-item orders with automatic volume discounts
- **Secure Payments**: STX-based payment processing with automatic escrow
- **Transaction Analytics**: Detailed purchase history and marketplace statistics
- **Administrative Controls**: Comprehensive marketplace management tools

### Key Benefits
- Transparent pricing with automatic volume discounts
- Real-time inventory management
- Secure, trustless transactions
- Detailed transaction history and analytics
- Flexible discount tier configuration

## Contract Architecture

### Data Structures

#### Product Registry
```clarity
marketplace-product-registry: {
  unique-product-identifier: uint,
  product-display-name: string-ascii 64,
  cost-per-individual-unit: uint,
  current-inventory-count: uint,
  publicly-available-for-purchase: bool
}
```

#### Discount Tiers
```clarity
bulk-purchase-discount-tiers: {
  unique-product-identifier: uint,
  minimum-quantity-for-tier: uint,
  percentage-discount-offered: uint,
  discount-tier-currently-active: bool
}
```

#### Purchase History
```clarity
customer-purchase-history: {
  purchasing-customer-address: principal,
  unique-transaction-identifier: uint,
  product-purchased-identifier: uint,
  total-units-in-transaction: uint,
  actual-amount-paid: uint,
  blockchain-height-at-purchase: uint,
  discount-percentage-received: uint
}
```

## Business Rules

### Constants and Limits
- Maximum product name length: 64 characters
- Maximum single order quantity: 1,000,000 units
- Maximum product unit price: 1,000,000,000 microSTX
- Maximum discount percentage: 100%
- Maximum items in bulk order: 10
- Minimum product name length: 1 character
- Minimum purchasable quantity: 1 unit
- Minimum acceptable unit price: 1 microSTX

### Error Codes
- `ERR-UNAUTHORIZED-OPERATION` (100): Caller lacks administrative privileges
- `ERR-DUPLICATE-PRODUCT-REGISTRATION` (101): Product already exists
- `ERR-REQUESTED-PRODUCT-NOT-AVAILABLE` (102): Product not found or unavailable
- `ERR-INVALID-DISCOUNT-CONFIGURATION` (103): Invalid discount parameters
- `ERR-UNACCEPTABLE-PRICE-VALUE` (104): Price outside acceptable range
- `ERR-PAYMENT-AMOUNT-INSUFFICIENT` (105): Insufficient payment provided
- `ERR-QUANTITY-EXCEEDS-LIMITS` (106): Requested quantity exceeds limits
- `ERR-ORDER-PROCESSING-FAILURE` (107): Order processing failed
- `ERR-MARKETPLACE-CURRENTLY-INACTIVE` (108): Marketplace is suspended
- `ERR-BULK-ORDER-CANNOT-BE-EMPTY` (109): Bulk order contains no items
- `ERR-PARAMETER-VALUE-INVALID` (110): Invalid parameter provided
- `ERR-PRODUCT-NAME-REQUIREMENTS-NOT-MET` (111): Product name validation failed
- `ERR-STOCK-LEVELS-INSUFFICIENT` (112): Insufficient inventory
- `ERR-DISCOUNT-TIER-CONFIGURATION-MISSING` (113): Discount tier not configured
- `ERR-BULK-ORDER-SIZE-LIMIT-EXCEEDED` (114): Too many items in bulk order

## Public Functions

### Product Management (Admin Only)

#### register-new-marketplace-product
```clarity
(register-new-marketplace-product 
  (descriptive-product-name (string-ascii 64)) 
  (price-per-unit uint) 
  (starting-inventory-amount uint))
```
Registers a new product in the marketplace with initial inventory and pricing.

#### update-product-inventory-levels
```clarity
(update-product-inventory-levels 
  (product-identifier uint) 
  (updated-stock-quantity uint))
```
Updates the inventory count for an existing product.

#### revise-product-unit-pricing
```clarity
(revise-product-unit-pricing 
  (product-identifier uint) 
  (new-price-per-unit uint))
```
Updates the unit price for an existing product.

#### disable-product-marketplace-availability
```clarity
(disable-product-marketplace-availability (product-identifier uint))
```
Removes a product from public availability while maintaining its data.

### Discount Management (Admin Only)

#### configure-bulk-purchase-discount-tier
```clarity
(configure-bulk-purchase-discount-tier 
  (target-product-identifier uint) 
  (required-minimum-quantity uint) 
  (offered-discount-percentage uint))
```
Sets up volume-based discount tiers for specific products.

#### deactivate-bulk-discount-tier
```clarity
(deactivate-bulk-discount-tier 
  (target-product-identifier uint) 
  (tier-minimum-quantity uint))
```
Removes a specific discount tier configuration.

### Purchase Functions

#### process-individual-product-purchase
```clarity
(process-individual-product-purchase 
  (target-product-identifier uint) 
  (desired-quantity uint))
```
Processes a single product purchase with automatic discount application.

#### process-comprehensive-bulk-purchase
```clarity
(process-comprehensive-bulk-purchase 
  (bulk-order-items (list 10 { unique-product-identifier: uint, requested-quantity: uint })))
```
Processes multiple product purchases in a single transaction.

### Administrative Functions

#### transfer-administrative-control
```clarity
(transfer-administrative-control (designated-new-administrator principal))
```
Transfers marketplace administration to a new principal.

#### temporarily-suspend-marketplace-operations
```clarity
(temporarily-suspend-marketplace-operations)
```
Temporarily suspends all marketplace operations.

#### restore-marketplace-operational-status
```clarity
(restore-marketplace-operational-status)
```
Restores marketplace operations after suspension.

#### withdraw-accumulated-marketplace-revenue
```clarity
(withdraw-accumulated-marketplace-revenue (requested-withdrawal-amount uint))
```
Allows admin to withdraw accumulated marketplace revenue.

## Read-Only Functions

### retrieve-complete-product-information
```clarity
(retrieve-complete-product-information (product-identifier uint))
```
Returns complete product details for a given product ID.

### lookup-applicable-discount-tier-information
```clarity
(lookup-applicable-discount-tier-information 
  (product-identifier uint) 
  (intended-purchase-quantity uint))
```
Returns applicable discount information for a quantity.

### calculate-comprehensive-order-pricing
```clarity
(calculate-comprehensive-order-pricing 
  (product-identifier uint) 
  (requested-purchase-quantity uint))
```
Calculates total pricing including applicable discounts.

### retrieve-customer-transaction-details
```clarity
(retrieve-customer-transaction-details (transaction-identifier uint))
```
Returns transaction details for the calling principal.

### retrieve-comprehensive-marketplace-statistics
```clarity
(retrieve-comprehensive-marketplace-statistics)
```
Returns overall marketplace metrics and statistics.

## Usage Examples

### Setting Up a Product
```clarity
;; Register a new product
(contract-call? .marketplace register-new-marketplace-product 
  "Premium Coffee Beans" 
  u1000000  ;; 1 STX per unit
  u500)     ;; 500 units initial stock

;; Configure bulk discount (10% off for 50+ units)
(contract-call? .marketplace configure-bulk-purchase-discount-tier 
  u1        ;; Product ID
  u50       ;; Minimum quantity
  u10)      ;; 10% discount
```

### Making a Purchase
```clarity
;; Single product purchase
(contract-call? .marketplace process-individual-product-purchase 
  u1        ;; Product ID
  u75)      ;; Quantity (qualifies for bulk discount)

;; Bulk order with multiple products
(contract-call? .marketplace process-comprehensive-bulk-purchase 
  (list 
    { unique-product-identifier: u1, requested-quantity: u50 }
    { unique-product-identifier: u2, requested-quantity: u25 }))
```

## Security Considerations

- Administrative functions are protected by access control
- All payments are processed through STX transfers
- Inventory validation prevents overselling
- Input validation prevents invalid parameter attacks
- Marketplace can be suspended in emergency situations

## Deployment Requirements

- Stacks blockchain environment
- Clarity smart contract runtime
- STX tokens for payment processing
- Administrative principal for marketplace management