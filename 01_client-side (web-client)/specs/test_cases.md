Test Cases (Baseline)

Remains the same - is executed identically across:
- client-side
- server-side
- CAPI

| Test ID | User Flow | Expected Events |
|---------|------------|-----------------|
| T01 | Open product page | view_item |
| T02 | Add product to cart | add_to_cart |
| T03 | Start checkout | begin_checkout |
| T04 | Complete a (test) purchase | purchase |
| T05 | Click hero CTA | hero_cta_click |
| T06 | Header navigation click | header_nav_click |
| T07 | Submit contact form | sureforms_submit |

Purpose: Establish consistent, repeatable test flow across all tracking variants.