# BuyLyst.co — Product Requirements & System Design Specification

## Version 1.1  
## Prepared For: BuyLyst.co  
## Document Type: Functional Product & Workflow Specification  
## Purpose: Complete Business-Level System Design Reference

---

# 1. Executive Summary

BuyLyst.co is a retail operations platform designed for trading card stores to manage the complete lifecycle of purchasing products from customers.

The platform enables:
- Public-facing buylist browsing
- Customer sell order creation
- In-store intake workflows
- Quick buy purchasing
- Inventory reconciliation
- Inventory export generation
- Pricing and quantity management
- Reporting and analytics
- Auditability and operational controls

The system is intended to support:
- Local game stores
- High-volume TCG retailers
- Multi-employee purchasing operations
- Structured and unstructured purchasing workflows

The platform prioritizes:
- Auditability
- Operational accountability
- Purchasing intelligence
- Inventory accuracy
- Flexible pricing workflows
- Employee permission separation

---

# 2. Core System Goals

The system must:

- Allow stores to purchase products from customers efficiently
- Support both remote and in-store customer selling
- Maintain strict financial audit trails
- Support employee and manager operational workflows
- Prevent accidental overbuying
- Support inventory reconciliation after payout
- Provide advanced reporting and analytics
- Scale to very large trading card catalogs
- Support future integrations with inventory/POS systems

---

# 3. High-Level Platform Overview

The platform consists of five primary operational systems:

| System | Purpose |
|---|---|
| Public Buylist | Customer-facing searchable purchasing catalog |
| Customer Cart System | Customer sell order management |
| Intake & Purchasing System | Employee/manager processing workflows |
| Inventory Reconciliation System | Inventory verification and export workflows |
| Administrative & Reporting System | Pricing, analytics, controls, configuration |

---

# 4. Supported Product Types

The platform must support:
- Individual trading cards
- Sealed products
- Accessories
- Bulk inventory
- Miscellaneous/custom items

Supported games include but are not limited to:
- Magic: The Gathering
- Yu-Gi-Oh!
- Pokémon
- One Piece
- Lorcana
- Future expandable categories

---

# 5. User Roles & Permissions

---

# 5.1 Guest

## Purpose
Anonymous in-store kiosk seller.

## Capabilities
- Browse public buylist
- Create temporary local carts
- Submit carts in-store
- Search products

## Restrictions
- No account management
- No remote access
- No historical cart access
- Limited to approved devices/kiosk mode

---

# 5.2 User

## Purpose
Standard customer account.

## Capabilities
- Register/login
- Authenticate via email/password or OAuth providers
- Browse buylist
- Search products
- Create remote sell carts
- Submit shipped or in-person orders
- View cart history
- View cart statuses
- Cancel eligible carts

## Restrictions
- Cannot modify pricing
- Cannot access operational tools
- Cannot bypass buylist restrictions

---

# 5.3 Employee

## Purpose
Front counter intake staff.

## Capabilities
- View submitted/approved carts
- Intake customer orders
- Modify quantities
- Modify conditions
- Add/remove products
- Add intake notes
- Process payouts using predefined pricing
- Cancel pending carts

## Restrictions
- Cannot override pricing
- Cannot perform Quick Buys
- Cannot alter buylist settings
- Cannot modify system settings

---

# 5.4 Inventory

## Purpose
Inventory reconciliation and export processing.

## Capabilities
- Review processed purchases
- Verify product identity
- Verify quantities
- Verify conditions
- Add inventory reconciliation corrections
- Generate inventory exports
- Finalize inventory-ready state

## Restrictions
- Cannot modify financial payout history
- Cannot alter pricing
- Cannot modify customer payout records

---

# 5.5 Manager

## Purpose
Operational purchasing authority.

## Capabilities
- All employee permissions
- Run Quick Buy workflow
- Override pricing
- Modify payout structures
- Modify credit bonuses during intake
- Configure buy prices
- Configure buy quantities
- View analytics/reports
- Manage buylist configuration
- Manage emergency disables

## Restrictions
- Cannot manage users
- Cannot modify system-level administrative settings

---

# 5.6 Admin

## Purpose
System administrator.

## Capabilities
- All manager permissions
- User management
- Role assignment
- Server/system configuration
- Branding configuration
- Authentication settings
- Global purchasing configuration

---

# 6. Authentication & Identity System

---

# 6.1 Authentication Methods

The platform must support:
- Email/password authentication
- OAuth authentication
- Persistent authenticated sessions
- JWT/session-based authorization
- Secure password storage

Supported OAuth providers must include:
- Google
- Facebook
- X (Twitter)

The authentication system should support future provider expansion.

---

# 6.2 Session & Active Cart Persistence

Authenticated users must be able to maintain a single active/open cart across sessions and devices until the cart is submitted, cancelled, or otherwise finalized.

The system should preserve:
- Cart contents
- Product selections
- Quantities
- Conditions
- User progress during cart construction

The exact implementation strategy for active cart persistence is intentionally unspecified and may evolve over time.

---

# 6.3 Kiosk Authentication Rules

Guest kiosk sessions should:
- operate without account authentication,
- be restricted to approved store devices,
- support temporary/local cart creation,
- expire automatically after inactivity or completion.

---

# 7. Public Buylist System

---

# 7.1 Purpose

Provide customers with a searchable view of products the store is actively purchasing.

---

# 7.2 Required Features

Customers must be able to:
- Search by name
- Search by set/group
- Search by category/game
- Filter by rarity
- Filter by subtype
- Filter by collector number
- View buy price
- View remaining quantity target
- View product details
- View product image

---

# 7.3 Product Availability Rules

Products may become unavailable due to:
- Quantity target reached
- Product disabled manually
- Group disabled
- Category disabled
- Emergency disable actions

---

# 8. Customer Cart System

---

# 8.1 Active Cart Rules

The system must support a persistent active customer cart experience.

Users should be able to:
- continue unfinished carts across sessions,
- return to incomplete carts,
- maintain in-progress product selections,
- modify carts until submission or cancellation.

The platform should ensure users do not unintentionally create multiple simultaneous active customer sell orders.

---

# 8.2 Cart Features

Customers must be able to:
- Add products
- Remove products
- Modify quantity
- Select condition
- Select dropoff type
- Submit cart
- Cancel eligible carts
- View historical carts

---

# 8.3 Cart States

| Status | Meaning |
|---|---|
| Active | Editable customer cart |
| Submitted | Awaiting review/intake |
| Approved | Locked pricing |
| Processed | Customer paid |
| Reviewed | Inventory reconciled |
| Completed | Export finalized |
| Cancelled | Terminated cart |

---

# 9. Intake & Purchasing System

---

# 9.1 Purpose

Allow employees/managers to process customer submissions into finalized purchases.

---

# 9.2 Intake Workflow

Staff must be able to:
- Retrieve customer cart
- Verify products
- Modify conditions
- Modify quantities
- Add/remove products
- Add intake notes
- Split payouts between cash/store credit
- Finalize purchase

---

# 9.3 Pricing Rules

Employees:
- Cannot modify pricing

Managers:
- Can override pricing
- Can override payout structures
- Can adjust credit bonuses

---

# 9.4 Condition-Based Pricing

The system must support:
- Near Mint
- Lightly Played
- Moderately Played
- Heavily Played
- Damaged

Condition modifiers are configurable globally.

---

# 10. Quick Buy System

---

# 10.1 Purpose

Allow managers to purchase products outside the structured public buylist workflow.

---

# 10.2 Quick Buy Capabilities

Managers must be able to:
- Search entire product catalog
- View product metadata
- View purchasing analytics
- Override prices
- Purchase products not currently buylisted
- Add custom/manual items

---

# 10.3 Product Metadata View

Must display:
- Product image
- Product name
- Game/category
- Set/group
- Rarity
- Number
- Market pricing data
- Current buylist status
- Remaining quantity targets
- Purchase history
- Historical buy velocity
- Historical average buy price

---

# 10.4 TCGPlayer Manual Review

Managers must be able to:
- Open product in TCGPlayer in a new tab
- Perform manual edge-case verification

---

# 10.5 Custom Item Support

Quick Buy must support non-catalog entries.

Examples:
- Bulk lots
- Miscellaneous accessories
- Damaged inventory
- Unknown products
- Temporary/manual items

Custom items must support:
- Custom name
- Custom description
- Quantity
- Pricing
- Notes

---

# 11. Inventory Reconciliation System

---

# 11.1 Purpose

Separate financial intake records from inventory truth.

---

# 11.2 Financial Record

Financial intake records represent:
- What customer was paid for
- Intake-side quantities
- Intake-side conditions
- Original payout values

Financial records become immutable once payout occurs.

---

# 11.3 Inventory Record

Inventory reconciliation records represent:
- Actual inventory received
- Corrected quantities
- Corrected conditions
- Product substitutions/corrections

---

# 11.4 Discrepancy Tracking

The system must track:
- Quantity discrepancies
- Condition discrepancies
- Product discrepancies
- Removed items
- Added items

---

# 11.5 Discrepancy Visibility

Users must be able to clearly see:
- Original intake values
- Inventory-corrected values
- Who made changes
- Why changes were made
- Timestamp of changes

---

# 11.6 Export System

Inventory users must be able to:
- Generate exports
- Export finalized inventory data
- Export to external systems
- Maintain export history

---

# 12. Buylist Configuration System

Managers/Admins must be able to:
- Configure buy prices
- Configure quantity targets
- Enable/disable products
- Configure percentage-based pricing
- Configure fixed-price overrides

---

# 13. Quantity & Progress Tracking

The system must track:
- Target quantity
- Pending quantity
- Purchased quantity
- Remaining quantity

The system must prevent:
- Purchasing above configured targets

---

# 14. Emergency Disable System

Managers/Admins must be able to:
- Disable categories
- Disable groups/sets
- Re-enable categories/groups

Emergency disables must immediately prevent:
- New cart additions
- New purchases

---

# 15. Reporting & Analytics System

---

# 15.1 Purpose

Provide operational purchasing intelligence.

---

# 15.2 Reporting Filters

Reports must support filtering by:
- Timeframe
- Category
- Group/set
- Product
- Employee
- Manager
- Condition
- Payout type
- Quick Buy vs Buylist
- Shipped vs in-person

---

# 15.3 Reporting Metrics

Reports must support:
- Total spent
- Quantity purchased
- Average purchase price
- Purchase velocity
- Condition breakdown
- Store credit issued
- Product profitability estimates
- Intake discrepancies
- Top purchased products
- Employee activity

---

# 15.4 Product-Level Analytics

When viewing products in admin/manager tools, the system should display:
- Historical purchase quantity
- Historical average buy price
- Purchase velocity charts
- Recent purchase history
- Remaining quantity targets
- Pending purchase totals

---

# 16. Audit Logging System

All write operations must be logged.

Audit logs must track:
- User performing action
- Timestamp
- Action type
- Entity modified
- Before/after state where applicable
- Reason/comments if applicable

Audit logs must support filtering/searching.

---

# 17. System Configuration

Admins must be able to configure:
- Store name
- Store branding/logo
- Credit bonus defaults
- Condition modifiers
- Enabled categories
- Registration availability
- OAuth provider configuration
- Dropoff options
- Kiosk settings

---

# 18. Non-Functional Requirements

---

# 18.1 Scalability

The platform must support:
- Large product catalogs
- High transaction volume
- Multiple simultaneous employees
- High-frequency searching/filtering

---

# 18.2 Auditability

The platform must preserve:
- Financial history
- User accountability
- Intake history
- Inventory correction history

---

# 18.3 Reliability

The system must:
- Prevent accidental financial modification
- Preserve immutable payout history
- Support operational recovery
- Prevent overspending due to race conditions

---

# 18.4 Usability

The platform should prioritize:
- Fast search workflows
- Counter-friendly interfaces
- Minimal employee friction
- Clear discrepancy visualization
- Efficient operational flows

---

# 19. Future Expansion Considerations

Potential future features:
- Multi-store support
- POS integrations
- Automated repricing
- Marketplace synchronization
- Customer reputation scoring
- AI-assisted grading assistance
- Mobile intake applications
- Barcode scanning
- Receipt/invoice generation
- Accounting exports
- Real-time market alerts

---

# 20. Conclusion

BuyLyst.co is designed to function as a complete retail purchasing operations platform for trading card stores.

The platform combines:
- customer-facing buylist workflows,
- operational purchasing tools,
- inventory reconciliation,
- reporting,
- and administrative controls

into a unified system focused on:
- scalability,
- accountability,
- operational intelligence,
- and financial accuracy.

This document represents the functional specification and operational design requirements for the platform and should serve as the primary reference for product planning, system architecture, UI/UX planning, backend design, and future expansion.
